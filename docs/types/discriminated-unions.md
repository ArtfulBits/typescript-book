### Discriminated Union

Якщо у вас є клас із[*literal member*](./literal-types.md), ви можете використовувати цю властивість для розрізнення членів об’єднання.

Як приклад розглянемо об’єднання `Square` і `Rectangle`, тут ми маємо член `kind` який існує в обох членах об’єднання та має певний *literal type*:

```ts
interface Square {
    kind: "square";
    size: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
type Shape = Square | Rectangle;
```

Якщо ви використовуєте перевірку стилю захисту типу (`==`, `===`, `!=`, `!==`) або `switch` на *discriminant property* (тут `kind`) TypeScript зрозуміє що об’єкт має бути типу, який має цей конкретний літерал, і додасть тип за вас :)

```ts
function area(s: Shape) {
    if (s.kind === "square") {
        // Зараз TypeScript *знає* що `s` має бути square ;)
        // Тож ви можете безпечно використовувати його учасників :)
        return s.size * s.size;
    }
    else {
        // Це не square? TypeScript зрозуміє, що це має бути Rectangle ;)
        // Тож ви можете безпечно використовувати його учасників :)
        return s.width * s.height;
    }
}
```

### Exhaustive Checks
Вичерпні перевірки.

Досить часто ви хочете переконатися, що всі члени мають певний код

```ts
interface Square {
    kind: "square";
    size: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
// Хтось щойно додав цей новий тип`Circle`
// Ми хотіли б дозволити TypeScript видавати помилку в будь-якому місці, яке *needs* обробити це

interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle;
```

Як приклад того, де все йде погано:

```ts
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    // Було б чудово, якби ви могли змусити TypeScript видавати вам помилку?
}
```

Ви можете зробити це, просто додавши пропуск і переконавшись, що виведений тип у цьому блоці сумісний із типом `never`. Наприклад, якщо ви додасте вичерпну перевірку, ви отримаєте гарну помилку:

```ts
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    else {
        // ERROR : `Circle` не може бути зведений до `never`
        const _exhaustiveCheck: never = s;
    }
}
```

Це змушує вас розглядати цю нову справу:

```ts
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    else if (s.kind === "circle") {
        return Math.PI * (s.radius **2);
    }
    else {
        // Okay once more
        const _exhaustiveCheck: never = s;
    }
}
```


### Switch
ПОРАДА: звичайно, ви також можете зробити це в операторі `switch`:

```ts
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.width * s.height;
        case "circle": return Math.PI * s.radius * s.radius;
        default: const _exhaustiveCheck: never = s;
    }
}
```

[references-discriminated-union]:https://github.com/Microsoft/TypeScript/pull/9163

### strictNullChecks

Якщо використовується *strictNullChecks* і виконуються вичерпні перевірки, TypeScript може скаржитися, що «не всі шляхи коду повертають значення». Ви можете заглушити це, просто повернувши змінну `_exhaustiveCheck` (типу `never`). Так:

```ts
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.width * s.height;
        case "circle": return Math.PI * s.radius * s.radius;
        default:
          const _exhaustiveCheck: never = s;
          return _exhaustiveCheck;
    }
}
```

### Throw in exhaustive checks
Ви можете написати функцію, яка приймає `never` (і тому може бути викликана лише зі змінною, яка виводиться як `never`), а потім видає помилку, якщо її тіло коли-небудь виконується:

```ts
function assertNever(x:never): never {
    throw new Error('Unexpected value. Should have been never.');
}
```

Приклад використання функції площі:

```ts
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
type Shape = Square | Rectangle;

function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.width * s.height;
     // Якщо під час компіляції додається новий випадок, ви отримаєте помилку компіляції
     // Якщо під час виконання з’являється нове значення, ви отримаєте помилку виконання
        default: return assertNever(s);
    }
}
```

### Retrospective Versioning
Скажімо, у вас є структура даних у формі:

```ts
type DTO = {
  name: string
}
```
І після того, як у вас є купа `DTO`, ви розумієте, що `name` було поганим вибором. Ви можете додати керування версіями ретроспективно, створивши новий *union* з *literal number* (або рядком, якщо хочете) DTO. Позначте версію 0 як `undefined` якщо у вас увімкнено *strictNullChecks*, це просто спрацює:

```ts
type DTO = 
| { 
   version: undefined, // version 0
   name: string,
 }
| {
   version: 1,
   firstName: string,
   lastName: string, 
}
// Even later 
| {
    version: 2,
    firstName: string,
    middleName: string,
    lastName: string, 
} 
// So on
```

Приклад використання такого DTO:

```ts
function printDTO(dto:DTO) {
  if (dto.version == null) {
      console.log(dto.name);
  } else if (dto.version == 1) {
      console.log(dto.firstName,dto.lastName);
  } else if (dto.version == 2) {
      console.log(dto.firstName, dto.middleName, dto.lastName);
  } else {
      const _exhaustiveCheck: never = dto;
  }
}
```

### Redux

Популярною бібліотекою, яка використовує це, є redux.

Ось [*gist of redux*](https://github.com/reactjs/redux#the-gist)із доданими анотаціями типу TypeScript:

```ts
import { createStore } from 'redux'

type Action
  = {
    type: 'INCREMENT'
  }
  | {
    type: 'DECREMENT'
  }
/**
* Це редʼюсер, чиста функція з (стан, дія) =>  стан.
  * Описує, як дія перетворює початковий стан на наступний.
  *
  * Форма стану залежить від вас: це може бути примітив, масив, об’єкт,
  * або навіть структуру даних Immutable.js. Єдина важлива частина полягає в тому, що ви повинні
  * не змінювати об'єкт стану, але повертати новий об'єкт, якщо стан змінюється.
  *
  * У цьому прикладі ми використовуємо оператор `switch` і рядки, але ви можете використовувати помічник який
  * дотримується іншої конвенції (наприклад, функція map), якщо це має сенс для вас
  * демонструвати.
  */
function counter(state = 0, action: Action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// Створення сховища Redux, що зберігає стан вашої програми.
// Його API — { subscribe, dispatch, getState }.
let store = createStore(counter)

// Ви можете використовувати subscribe() для оновлення інтерфейсу користувача у відповідь на зміни стану.
// Зазвичай ви використовуєте бібліотеку прив’язки перегляду (наприклад, React Redux), а не subscribe() безпосередньо.
// Однак також може бути зручно зберегти поточний стан у localStorage.

store.subscribe(() =>
  console.log(store.getState())
)

// Єдиний спосіб змінити внутрішній стан - це відправити action.
// Аction можна серіалізувати, реєструвати або зберігати, а потім відтворювати.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

Використання його з TypeScript забезпечує захист від друкарських помилок, покращує здатність до рефакторингу та самодокументування коду.

