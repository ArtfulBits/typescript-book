## readonly
Система типів TypeScript дозволяє відмічати окремі властивості інтерфейсу як `readonly`. Це дозволяє працювати у функціональному стилі (непередбачувана мутація - погано):

```ts
function foo(config: {
    readonly bar: number,
    readonly bas: number
}) {
    // ..
}

let config = { bar: 123, bas: 123 };
foo(config);
// Ви можете бути впевнені, що `config` не змінено 🌹
```

Звичайно, ви також можете використовувати `readonly` в описах `interface` та `type`, наприклад:

```ts
type Foo = {
    readonly bar: number;
    readonly bas: number;
}

// Ініціалізація допустима
let foo: Foo = { bar: 123, bas: 456 };

// Мутація неможлива
foo.bar = 456; // Помилка: Ліва частина виразу присвоєння не може бути константою або властивістю тільки для читання
```

Ви навіть можете оголосити властивість класу як `readonly`. Ви можете ініціалізувати їх на момент оголошення або в конструкторі, як показано нижче:

```ts
class Foo {
    readonly bar = 1; // OK
    readonly baz: string;
    constructor() {
        this.baz = "hello"; // OK
    }
}
```

## Readonly
Є тип `Readonly`, який приймає тип `T` та відмічає всі його властивості як `readonly`, використовуючи відображені типи. Ось демонстраційний приклад, який використовує його на практиці:

```ts
type Foo = {
  bar: number;
  bas: number;
}

type FooReadonly = Readonly<Foo>; 

let foo: Foo = {bar: 123, bas: 456};
let fooReadonly: FooReadonly = {bar: 123, bas: 456};

foo.bar = 456; // Добре
fooReadonly.bar = 456; // ПОМИЛКА: bar є тільки для читання
```

### Різноманітні використання

#### ReactJS
Одна з бібліотек, яка любить незмінність - це ReactJS, ви *можете* відмітити ваші `Props` та `State` як незмінні, наприклад:

```ts
interface Props {
    readonly foo: number;
}
interface State {
    readonly bar: number;
}
export class Something extends React.Component<Props,State> {
  someMethod() {
    // Ви можете бути впевнені, що ніхто не зробить
    this.props.foo = 123; // ПОМИЛКА: (props є незмінними)
    this.state.baz = 456; // ПОМИЛКА: (треба використовувати this.setState)  
  }
}
```

Проте вам не потрібно цього робити, оскільки типові описи для React вже відмічають їх як `readonly` (внутрішньо обгортаючи передані загальні типи згаданим вище типом `Readonly`).

```ts
export class Something extends React.Component<{ foo: number }, { baz: number }> {
  // Ви можете бути впевнені, що ніхто не зробить
  someMethod() {
    this.props.foo = 123; // ПОМИЛКА: (props є незмінними)
    this.state.baz = 456; // ПОМИЛКА: (треба використовувати this.setState)  
  }
}
```

#### Seamless Immutable

Ви навіть можете відмітити підписи індексів як тільки для читання:

```ts
/**
 * Оголошення
 */
interface Foo {
    readonly[x: number]: number;
}

/**
 * Використання
 */
let foo: Foo = { 0: 123, 2: 345 };
console.log(foo[0]);   // Добре (читання)
foo[0] = 456;          // Помилка (мутація): тільки для читання
```

Це чудово, якщо ви хочете використовувати масиви JavaScript у *незмінному* стилі. Насправді, TypeScript поставляється з інтерфейсом `ReadonlyArray<T>`, щоб дозволити вам робити саме це:

```ts
let foo: ReadonlyArray<number> = [1, 2, 3];
console.log(foo[0]);   // Добре
foo.push(4);           // Помилка: `push` не існує в ReadonlyArray, оскільки він змінює масив
foo = foo.concat([4]); // Добре: створити копію
```

#### Автоматичне виведення
У деяких випадках компілятор може автоматично вивести певний елемент як тільки для читання, наприклад, у класі, якщо у властивості є тільки геттер, але немає сеттера, вона вважається тільки для читання, наприклад:

```ts
class Person {
    firstName: string = "John";
    lastName: string = "Doe";
    get fullName() {
        return this.firstName + this.lastName;
    }
}

const person = new Person();
console.log(person.fullName); // John Doe
person.fullName = "Dear Reader"; // Помилка! fullName є тільки для читання
```

### Відмінність від `const`
`const`

1. для змінної посилання
1. змінну неможливо переприсвоїти на щось інше.

`readonly` є

1. для властивості
1. властивість може бути змінена через псевдоніми

Приклад, що пояснює 1:

```ts
const foo = 123; // посилання на змінну
var bar: {
    readonly bar: number; // для властивості
}
```

Приклад, що пояснює 2:

```ts
let foo: {
    readonly bar: number;
} = {
        bar: 123
    };

function iMutateFoo(foo: { bar: number }) {
    foo.bar = 456;
}

iMutateFoo(foo); // Аргумент foo має псевдонім параметра foo
console.log(foo.bar); // 456!
```

По суті, `readonly` забезпечує, що властивість *не може бути змінена мною*, але якщо ви дасте її комусь, хто не має такої гарантії (дозволено з причин сумісності типів), вони можуть її змінити. Звичайно, якщо `iMutateFoo` заявив, що вони не змінюють `foo.bar`, компілятор правильно позначив би це як помилку, як показано нижче:

```ts
interface Foo {
    readonly bar: number;
}
let foo: Foo = {
    bar: 123
};

function iTakeFoo(foo: Foo) {
    foo.bar = 456; // Помилка! bar є тільки для читання
}

iTakeFoo(foo); // Аргумент foo має псевдонім параметра foo
```

[](https://github.com/Microsoft/TypeScript/pull/6532)