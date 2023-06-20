## readonly
Система типів TypeScript дозволяє позначати окремі властивості інтерфейсу як `readonly`. Це дозволяє вам працювати функціонально (неочікувана мутація погана):

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

Звичайно, ви також можете використовувати `readonly` у визначеннях `interface` і `type`, наприклад:

```ts
type Foo = {
    readonly bar: number;
    readonly bas: number;
}

// Initialization is okay
let foo: Foo = { bar: 123, bas: 456 };

// Mutation is not
foo.bar = 456; // Error: Left-hand side of assignment expression cannot be a constant or a read-only property
```

Ви навіть можете оголосити властивість класу як `readonly`. Ви можете ініціалізувати їх у точці оголошення або в конструкторі, як показано нижче:

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
Існує тип `Readonly` який приймає тип `Т` і позначає всі його властивості як `readonly` за допомогою зіставлених типів. Ось демо, яке використовує це на практиці:

```ts
type Foo = {
  bar: number;
  bas: number;
}

type FooReadonly = Readonly<Foo>; 

let foo: Foo = {bar: 123, bas: 456};
let fooReadonly: FooReadonly = {bar: 123, bas: 456};

foo.bar = 456; // Okay
fooReadonly.bar = 456; // ERROR: bar is readonly
```

### Various Use Cases

#### ReactJS
Однією з бібліотек, яка любить незмінність, є ReactJS, ви  *можете* позначити свої `Props` і `State` як незмінні, наприклад:

```ts
interface Props {
    readonly foo: number;
}
interface State {
    readonly bar: number;
}
export class Something extends React.Component<Props,State> {
  someMethod() {
    // You can rest assured no one is going to do
    this.props.foo = 123; // ERROR: (props are immutable)
    this.state.baz = 456; // ERROR: (one should use this.setState)  
  }
}
```

Однак вам не потрібно, оскільки визначення типів для React уже позначають їх як `readonly` (шляхом внутрішнього обгортання переданих загальних типів у тип `Readonly`, згаданий вище).

```ts
export class Something extends React.Component<{ foo: number }, { baz: number }> {
  // You can rest assured no one is going to do
  someMethod() {
    this.props.foo = 123; // ERROR: (props are immutable)
    this.state.baz = 456; // ERROR: (one should use this.setState)  
  }
}
```

#### Seamless Immutable

Ви навіть можете позначити підписи індексу як лише для читання:

```ts
/**
 * Declaration
 */
interface Foo {
    readonly[x: number]: number;
}

/**
 * Usage
 */
let foo: Foo = { 0: 123, 2: 345 };
console.log(foo[0]);   // Okay (reading)
foo[0] = 456;          // Error (mutating): Readonly
```

Це чудово, якщо ви хочете використовувати рідні масиви JavaScript у *незмінному* вигляді. Насправді TypeScript постачається з інтерфейсом `ReadonlyArray<T>`, який дозволяє вам робити саме це:

```ts
let foo: ReadonlyArray<number> = [1, 2, 3];
console.log(foo[0]);   // Okay
foo.push(4);           // Error: `push` does not exist on ReadonlyArray as it mutates the array
foo = foo.concat([4]); // Okay: create a copy
```

#### Automatic Inference
У деяких випадках компілятор може автоматично зробити висновок, що певний елемент є лише для читання, наприклад. у класі, якщо у вас є властивість, яка має лише геттер, але не сеттер, передбачається, що вона лише для читання, наприклад:

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
person.fullName = "Dear Reader"; // Error! fullName is readonly
```

### Difference from `const`
Різниця з `const`
`const`

1. для посилання на змінну
1. змінна не може бути перепризначена ні на що інше.

`readonly`

1. для властивості класу
1. властивість можна змінити через псевдонім

Зразок пояснення 1:

```ts
const foo = 123; // variable reference
var bar: {
    readonly bar: number; // for property
}
```

Зразок пояснення 2:

```ts
let foo: {
    readonly bar: number;
} = {
        bar: 123
    };

function iMutateFoo(foo: { bar: number }) {
    foo.bar = 456;
}

iMutateFoo(foo); // The foo argument is aliased by the foo parameter
console.log(foo.bar); // 456!
```

По суті, `readonly` гарантує, що властивість *не може буди змінена самостійно*, але якщо ви надасте її комусь, хто не має такої гарантії (дозволено з причин сумісності типів), вони можуть змінити її. Звичайно, якщо `iMutateFoo` сказав, що вони не змінюють `foo.bar`, компілятор правильно позначить це як помилку, як показано:

```ts
interface Foo {
    readonly bar: number;
}
let foo: Foo = {
    bar: 123
};

function iTakeFoo(foo: Foo) {
    foo.bar = 456; // Error! bar is readonly
}

iTakeFoo(foo); // The foo argument is aliased by the foo parameter
```

[](https://github.com/Microsoft/TypeScript/pull/6532)
