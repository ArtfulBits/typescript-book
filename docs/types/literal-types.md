## Літерали
Літерали - це *точні* значення, які є примітивами JavaScript.

### Літерали рядків

Ви можете використовувати літерал рядка як тип. Наприклад:

```ts
let foo: 'Hello';
```

Тут ми створили змінну під назвою `foo`, яка *дозволить призначати лише літеральне значення `'Hello'`*. Це демонструється нижче:

```ts
let foo: 'Hello';
foo = 'Bar'; // Помилка: "Bar" не може бути призначено типу "Hello"
```

Вони не дуже корисні самі по собі, але можуть бути поєднані в об'єднання типів, щоб створити потужну (і корисну) абстракцію, наприклад:

```ts
type CardinalDirection =
    | "North"
    | "East"
    | "South"
    | "West";

function move(distance: number, direction: CardinalDirection) {
    // ...
}

move(1,"North"); // Ок
move(1,"Nurth"); // Помилка!
```

### Інші літеральні типи
TypeScript також підтримує літеральні типи `boolean` та `number`, наприклад:

```ts
type OneToFive = 1 | 2 | 3 | 4 | 5;
type Bools = true | false;
```

### Виведення типу
Досить часто ви отримуєте помилку, як от `Type string is not assignable to type "foo"`. Наступний приклад демонструє це.

```js
function iTakeFoo(foo: 'foo') { }
const test = {
  someProp: 'foo'
};
iTakeFoo(test.someProp); // Помилка: Аргумент типу string не може бути призначений параметру типу 'foo'
```

Це тому, що `test` виводиться як тип `{someProp: string}`. Виправлення тут полягає в використанні простого підтвердження типу, щоб повідомити TypeScript про літерал, який ви хочете вивести, як показано нижче:

```js
function iTakeFoo(foo: 'foo') { }
const test = {
  someProp: 'foo' as 'foo'
};
iTakeFoo(test.someProp); // Ок!
```

або використовуйте анотацію типу, яка допомагає TypeScript вивести правильну річ у точці оголошення:

```ts
function iTakeFoo(foo: 'foo') { }
type Test = {
  someProp: 'foo',
}
const test: Test = { // Анотуйте - виведено, що someProp завжди === 'foo'
  someProp: 'foo' 
}; 
iTakeFoo(test.someProp); // Ок!
```

### Використання
Дійсні випадки використання літеральних типів рядків:

#### Enums на основі рядків

[Enums TypeScript базуються на числах](../enums.md). Ви можете використовувати літерали рядків з об'єднаними типами, щоб імітувати enums на основі рядків, як ми зробили в прикладі `CardinalDirection` вище. Ви навіть можете створити структуру `Ключ: Значення`, використовуючи наступну функцію:

```ts
/** Функція-утиліта для створення K:V зі списку рядків */
function strEnum<T extends string>(o: Array<T>): {[K in T]: K} {
  return o.reduce((res, key) => {
    res[key] = key;
    return res;
  }, Object.create(null));
}
```

А потім створити об'єднання типів літералів, використовуючи `keyof typeof`. Ось повний приклад:

```ts
/** Функція-утиліта для створення K:V зі списку рядків */
function strEnum<T extends string>(o: Array<T>): {[K in T]: K} {
  return o.reduce((res, key) => {
    res[key] = key;
    return res;
  }, Object.create(null));
}

/**
  * Приклад створення рядкового enum
  */

/** Створіть K:V */
const Direction = strEnum([
  'North',
  'South',
  'East',
  'West'
])
/** Створіть тип */
type Direction = keyof typeof Direction;

/** 
  * Приклад використання рядкового enum
  */
let sample: Direction;

sample = Direction.North; // Ок
sample = 'North'; // Ок
sample = 'AnythingElse'; // ПОМИЛКА!
```

#### Моделювання існуючих JavaScript API

Наприклад, [редактор CodeMirror має опцію `readOnly`](https://codemirror.net/doc/manual.html#option_readOnly), яка може бути або `boolean`, або літеральним рядком `"nocursor"` (ефективні дійсні значення `true`, `false`, `"nocursor"`). Це можна оголосити як:

```ts
readOnly: boolean | 'nocursor';
```

#### Розпізнавані об'єднання

Ми розглянемо [це пізніше в книзі](./discriminated-unions.md).


[](https://github.com/Microsoft/TypeScript/pull/5185)