## Literals
Літерали — це *точні* значення, які є примітивами JavaScript.

### String Literals

Ви можете використовувати рядковий літерал як тип. Наприклад:
```ts
let foo: 'Hello';
```

Тут ми створили змінну під назвою `foo`, яка *дозволить лише призначити йому літеральне значення `'Hello'`*. Це показано нижче:

```ts
let foo: 'Hello';
foo = 'Bar'; // Error: "Bar" не можна призначити типу "Hello"
```

Вони не дуже корисні самі по собі, але їх можна об’єднати в об’єднання типів для створення потужної (і корисної) абстракції, наприклад:

```ts
type CardinalDirection =
    | "North"
    | "East"
    | "South"
    | "West";

function move(distance: number, direction: CardinalDirection) {
    // ...
}

move(1,"North"); // Okay
move(1,"Nurth"); // Error!
```

### Other literal types
TypeScript також підтримує типи літералів `boolean` і `number`, наприклад:

```ts
type OneToFive = 1 | 2 | 3 | 4 | 5;
type Bools = true | false;
```

### Inference 
Досить часто ви отримуєте помилку на зразок `Тип string не можна призначити типу "foo"`. Наступний приклад демонструє це.

```js
function iTakeFoo(foo: 'foo') { }
const test = {
  someProp: 'foo'
};
iTakeFoo(test.someProp); // Error: Argument of type string is not assignable to parameter of type 'foo'
```

Це тому, що `test` має тип `{someProp: string}`. Виправлення тут полягає у використанні простого твердження типу, щоб повідомити TypeScript літерал, який ви хочете, щоб він визнав, як показано нижче:

```js
function iTakeFoo(foo: 'foo') { }
const test = {
  someProp: 'foo' as 'foo'
};
iTakeFoo(test.someProp); // Okay!
```

або використовуйте анотацію типу, яка допомагає TypeScript вивести правильний висновок у точці оголошення:

```ts
function iTakeFoo(foo: 'foo') { }
type Test = {
  someProp: 'foo',
}
const test: Test = { // Annotate - inferred someProp is always === 'foo'
  someProp: 'foo' 
}; 
iTakeFoo(test.someProp); // Okay!
```

### Use cases
Допустимі варіанти використання типів рядкових літералів:

#### String based enums

[TypeScript enums are number based](../enums.md). Ви можете використовувати рядкові літерали з типами об’єднання, щоб імітувати перелік на основі рядків, як ми робили у прикладі `CardinalDirection` вище. Ви навіть можете створити структуру `Key:Value` за допомогою такої функції:

```ts
/** Utility function to create a K:V from a list of strings */
function strEnum<T extends string>(o: Array<T>): {[K in T]: K} {
  return o.reduce((res, key) => {
    res[key] = key;
    return res;
  }, Object.create(null));
}
```

А потім згенеруйте літеральне об’єднання типів за допомогою `keyof typeof`. Ось повний приклад:

```ts
/** Utility function to create a K:V from a list of strings */
function strEnum<T extends string>(o: Array<T>): {[K in T]: K} {
  return o.reduce((res, key) => {
    res[key] = key;
    return res;
  }, Object.create(null));
}

/**
  * Sample create a string enum
  */

/** Create a K:V */
const Direction = strEnum([
  'North',
  'South',
  'East',
  'West'
])
/** Create a Type */
type Direction = keyof typeof Direction;

/** 
  * Sample using a string enum
  */
let sample: Direction;

sample = Direction.North; // Okay
sample = 'North'; // Okay
sample = 'AnythingElse'; // ERROR!
```

#### Modelling existing JavaScript APIs

наприклад [CodeMirror editor has an option `readOnly`](https://codemirror.net/doc/manual.html#option_readOnly), яка може бути або `логічним значенням`, або літеральним рядком `"nocursor"` (дійсні дійсні значення `true ,false,"nocursor"`). Його можна оголосити так:

```ts
readOnly: boolean | 'nocursor';
```

#### Discriminated Unions

Ми розглянемо [this later in the book](./discriminated-unions.md).


[](https://github.com/Microsoft/TypeScript/pull/5185)
