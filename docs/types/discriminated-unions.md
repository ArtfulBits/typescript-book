### Дискримінований об'єднаний тип

Якщо у вас є клас з літеральним членом, то ви можете використовувати цей властивість для розрізнення між членами об'єднання.

Наприклад, розгляньте об'єднання `Square` та `Rectangle`, тут ми маємо член `kind`, який існує на обох членах об'єднання та є певним *літеральним типом*:

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

Якщо ви використовуєте перевірку стилю типу охоронця (`==`, `===`, `!=`, `!==`) або `switch` на *відмінній властивості* (тут `kind`), TypeScript зрозуміє, що об'єкт повинен бути типу, який має цей конкретний літерал, і зробить для вас звуження типу :)

```ts
function area(s: Shape) {
    if (s.kind === "square") {
        // Тепер TypeScript *знає*, що `s` повинен бути квадратом ;)
        // Тож ви можете безпечно використовувати його члени :)
        return s.size * s.size;
    }
    else {
        // Це не був квадрат? Тоді TypeScript зрозуміє, що це повинен бути прямокутник ;)
        // Тож ви можете безпечно використовувати його члени :)
        return s.width * s.height;
    }
}
```

### Вичерпні перевірки
Досить часто ви хочете переконатися, що всі члени об'єднання мають якийсь код (дію) проти них.

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

// Хтось тільки що додав цей новий тип `Circle`
// Ми хотіли б, щоб TypeScript дав помилку в будь-якому місці, де *потрібно* звернути увагу на це
interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle;
```

Наприклад, деякі речі йдуть погано:

```ts
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    // Чи не було б чудово, якби ви могли отримати від TypeScript помилку?
}
```

Ви можете зробити це, просто додавши падіння і переконавшись, що виведений тип в тому блоку сумісний з типом `never`. Наприклад, якщо ви додаєте вичерпну перевірку, ви отримуєте гарну помилку:

```ts
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    else {
        // ПОМИЛКА: `Circle` не може бути призначений для `never`
        const _exhaustiveCheck: never = s;
    }
}
```

Це змушує вас обробляти цей новий випадок:

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
        // Окей, ще раз
        const _exhaustiveCheck: never = s;
    }
}
```


### Перемикач
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

Якщо використовується *strictNullChecks* та робляться вичерпні перевірки, TypeScript може скаржитися на "не всі шляхи коду повертають значення". Ви можете заткнути це, просто повернувши змінну `_exhaustiveCheck` (типу `never`). Отже:

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

### Помилки в вичерпних перевірках
Ви можете написати функцію, яка приймає `never` (і тому може бути викликана тільки змінною, яка виводиться як `never`) і потім генерує помилку, якщо її тіло коли-небудь виконується: 

```ts
function assertNever(x:never): never {
    throw new Error('Unexpected value. Should have been never.');
}
```

Приклад використання з функцією area: 

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
		// Якщо на етапі компіляції додається новий випадок, ви отримаєте помилку компіляції
		// Якщо на етапі виконання з'являється нове значення, ви отримаєте помилку виконання
        default: return assertNever(s);
    }
}
```

### Ретроспективне версіювання
Скажімо, у вас є структура даних наступного вигляду: 

```ts
type DTO = {
  name: string
}
```
І після того, як у вас є купа `DTO`, ви розумієте, що `name` було поганим вибором. Ви можете додати версіювання ретроспективно, створивши нове *об'єднання* з *літеральним числом* (або рядком, якщо ви хочете) DTO. Позначте версію 0 як `undefined`, і якщо у вас увімкнено *strictNullChecks*, це просто працюватиме: 

```ts
type DTO = 
| { 
   version: undefined, // версія 0
   name: string,
 }
| {
   version: 1,
   firstName: string,
   lastName: string, 
}
// Ще пізніше 
| {
    version: 2,
    firstName: string,
    middleName: string,
    lastName: string, 
} 
// Так далі
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

Популярна бібліотека, яка використовує це - це redux.

Ось [*gist of redux*](https://github.com/reactjs/redux#the-gist) з додаванням анотацій типів TypeScript:

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
 * Це редуктор, чиста функція з підписом (state, action) => state.
 * Він описує, як дія перетворює стан на наступний стан.
 *
 * Форма держави залежить від вас: це може бути примітивом, масивом, об'єктом,
 * або навіть структурою даних Immutable.js. Єдине важливе - ви не повинні
 * змінювати об'єкт стану, але повертати новий об'єкт, якщо стан змінюється.
 *
 * У цьому прикладі ми використовуємо оператор `switch` та рядки, але ви можете використовувати допоміжний засіб, який
 * слідує іншій конвенції (наприклад, картам функцій), якщо це має сенс для вашого
 * проект.
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

// Створіть сховище Redux, яке містить стан вашого додатка.
// Його API - { підписатися, відправити, отриматиState }.
let store = createStore(counter)

// Ви можете використовувати subscribe(), щоб оновлювати інтерфейс користувача відповідно до змін стану.
// Зазвичай ви використовували бібліотеку зв'язування представлення (наприклад, React Redux), а не підписатися() безпосередньо.
// Однак також може бути зручно зберігати поточний стан в localStorage.

store.subscribe(() =>
  console.log(store.getState())
)

// Єдиний спосіб змінити внутрішній стан - це відправити дію.
// Дії можна серіалізувати, реєструвати або зберігати та відтворювати пізніше.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

Використання TypeScript дає вам безпеку від помилок друка, збільшується можливість рефакторингу та самодокументуючий код.