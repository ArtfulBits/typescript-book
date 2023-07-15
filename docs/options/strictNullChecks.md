# `strictNullChecks`

За замовчуванням `null` і `undefined` можна призначити всім типам у TypeScript, наприклад,

```ts
let foo: number = 123;
foo = null; // Okay
foo = undefined; // Okay
```

Це змодельовано за тим, як багато людей пишуть JavaScript. Однак, як і всі інші речі, TypeScript дозволяє *явно* визначити, чому *can and cannot be* присвоїти значення `null` або `undefined`.

У режимі суворої перевірки null `null` і `undefined` відрізняються:

```ts
let foo = undefined;
foo = null; // NOT Okay
```

Скажімо, у нас є інтерфейс `Member`:

```ts
interface Member {
  name: string,
  age?: number
}
```

Не кожен `Member` вказує свій вік, тому `age` є необов'язковою властивістю, тобто значення `age` може бути або не бути `undefined`.

`undefined` - це корінь усього зла. Це часто призводить до помилок виконання. Легко написати код, який видаватиме `Error` під час виконання:

```ts
getMember()
  .then(member: Member => {
    const stringifyAge = member.age.toString() // не можу примінити 'toString' к undefined
  })
```

Але в режимі суворої перевірки null ця помилка буде виявлена під час компіляції:

```ts
getMember()
  .then(member: Member => {
    const stringifyAge = member.age.toString() // Об'єкт можливо 'undefined'
  })
```

## Non-Null Assertion Operator

Новий оператор постфіксованого виразу `!` може бути використаний для підтвердження того, що його операнд не є нульовим і не визначеним у контекстах, де перевірка типу не може зробити висновок про цей факт. Наприклад:

```ts
// Compiled with --strictNullChecks
function validateEntity(e?: Entity) {
    // Throw exception if e is null or invalid entity
}

function processEntity(e?: Entity) {
    validateEntity(e);
    let a = e.name;  // TS ERROR: e may be null.
    let b = e!.name;  // OKAY. We are asserting that e is non-null.
}
```

> Зауважте, що це лише твердження, і, як і твердження типу *ви відповідаєте* за те, щоб значення не було нульовим. Ненульове твердження, по суті, означає, що ви говорите компілятору: «Я знаю, що це не нуль, тому дозвольте мені використовувати його так, ніби воно не нульове».

### Definite Assignment Assertion Operator

TypeScript також скаржиться на неініціалізацію властивостей у класах, наприклад:

```ts
class C {
  foo: number; // OKAY as assigned in constructor
  bar: string = "hello"; // OKAY as has property initializer
  baz: boolean; // TS ERROR: Property 'baz' has no initializer and is not assigned directly in the constructor.
  constructor() {
    this.foo = 42;
  }
}
```

Ви можете використовувати твердження визначеного призначення, додане до назви властивості, щоб повідомити TypeScript, що ви ініціалізуєте його десь, крім конструктора, наприклад.

```ts
class C {
  foo!: number;
  // ^
  // Notice this exclamation point!
  // This is the "definite assignment assertion" modifier.
  
  constructor() {
    this.initialize();
  }
  initialize() {
    this.foo = 0;
  }
}
```

Ви також можете використовувати це твердження з простими оголошеннями змінних, наприклад:

```ts
let a: number[]; // No assertion
let b!: number[]; // Assert

initialize();

a.push(4); // TS ERROR: variable used before assignment
b.push(4); // OKAY: because of the assertion

function initialize() {
  a = [0, 1, 2, 3];
  b = [0, 1, 2, 3];
}
```

> Як і всі твердження, ви говорите компілятору довіряти вам. Компілятор не скаржиться, навіть якщо код насправді не завжди призначає властивість.
