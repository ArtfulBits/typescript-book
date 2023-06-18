* [Type Guard](#type-guard)
* [User Defined Type Guards](#user-defined-type-guards)

## Type Guard
Захисники типу дозволяють звузити тип об’єкта в межах умовного блоку.


### typeof

TypeScript знає про використання операторів `instanceof` і `typeof` JavaScript. Якщо ви використовуєте їх в умовному блоці, TypeScript зрозуміє, що тип змінної буде іншим у цьому умовному блоці. Ось короткий приклад, коли TypeScript розуміє, що певна функція не існує в `string`, і вказує на те, що, ймовірно, було помилкою користувача:

```ts
function doSomething(x: number | string) {
    if (typeof x === 'string') { // Within the block TypeScript knows that `x` must be a string
        console.log(x.subtr(1)); // Error, 'subtr' does not exist on `string`
        console.log(x.substr(1)); // OK
    }
    x.substr(1); // Error: There is no guarantee that `x` is a `string`
}
```

### instanceof

Ось приклад із класом і `instanceof`:

```ts
class Foo {
    foo = 123;
    common = '123';
}

class Bar {
    bar = 123;
    common = '123';
}

function doStuff(arg: Foo | Bar) {
    if (arg instanceof Foo) {
        console.log(arg.foo); // OK
        console.log(arg.bar); // Error!
    }
    if (arg instanceof Bar) {
        console.log(arg.foo); // Error!
        console.log(arg.bar); // OK
    }

    console.log(arg.common); // OK
    console.log(arg.foo); // Error!
    console.log(arg.bar); // Error!
}

doStuff(new Foo());
doStuff(new Bar());
```

TypeScript навіть розуміє `else`, тому, коли `if` звужує один тип, він знає, що всередині else *it's definitely not that type*. Ось приклад:

```ts
class Foo {
    foo = 123;
}

class Bar {
    bar = 123;
}

function doStuff(arg: Foo | Bar) {
    if (arg instanceof Foo) {
        console.log(arg.foo); // OK
        console.log(arg.bar); // Error!
    }
    else {  // MUST BE Bar!
        console.log(arg.foo); // Error!
        console.log(arg.bar); // OK
    }
}

doStuff(new Foo());
doStuff(new Bar());
```

### in 

Оператор `in` виконує безпечну перевірку наявності властивості об'єкта та може використовуватися як захист типу. наприклад

```ts
interface A {
  x: number;
}
interface B {
  y: string;
}

function doStuff(q: A | B) {
  if ('x' in q) {
    // q: A
  }
  else {
    // q: B
  }
}
```

### Literal Type Guard

Ви можете використовувати `===` / `==` / `!==` / `!=`, щоб розрізняти літеральні значення

```ts
type TriState = 'yes' | 'no' | 'unknown';

function logOutState(state:TriState) {
  if (state == 'yes') {
    console.log('User selected yes');
  } else if (state == 'no') {
    console.log('User selected no');
  } else {
    console.log('User has not made a selection yet');
  }
}
```

Це навіть працює, коли у вас є буквальні типи в об’єднанні. Ви можете перевірити значення назви спільної властивості, щоб розрізнити об’єднання, наприклад.

```ts
type Foo = {
  kind: 'foo', // Literal type 
  foo: number
}
type Bar = {
  kind: 'bar', // Literal type 
  bar: number
}

function doStuff(arg: Foo | Bar) {
    if (arg.kind === 'foo') {
        console.log(arg.foo); // OK
        console.log(arg.bar); // Error!
    }
    else {  // MUST BE Bar!
        console.log(arg.foo); // Error!
        console.log(arg.bar); // OK
    }
}
```

### null and undefined with `strictNullChecks`

TypeScript достатньо розумний, щоб виключити як `null`, так і `undefined` за допомогою перевірки `== null` / `!= null`. Наприклад:

```ts
function foo(a?: number | null) {
  if (a == null) return;

  // a is number now.
}
```

### User Defined Type Guards
У JavaScript не надто багато вбудованої підтримки інтроспекції під час виконання. Коли ви використовуєте звичайні об’єкти JavaScript (використовуючи структурну типізацію для вашої переваги), ви навіть не маєте доступу до `instanceof` або `typeof`. Для цих випадків ви можете створити *User Defined Type Guard functions*. Це лише функції, які повертають `someArgumentName is SomeType`. Ось приклад:

```ts
/**
 * Just some interfaces
 */
interface Foo {
    foo: number;
    common: string;
}

interface Bar {
    bar: number;
    common: string;
}

/**
 * User Defined Type Guard!
 */
function isFoo(arg: any): arg is Foo {
    return arg.foo !== undefined;
}

/**
 * Sample usage of the User Defined Type Guard
 */
function doStuff(arg: Foo | Bar) {
    if (isFoo(arg)) {
        console.log(arg.foo); // OK
        console.log(arg.bar); // Error!
    }
    else {
        console.log(arg.foo); // Error!
        console.log(arg.bar); // OK
    }
}

doStuff({ foo: 123, common: '123' });
doStuff({ bar: 123, common: '123' });
```

### Type Guards and callbacks

TypeScript не передбачає, що guards залишаються активними у зворотних викликах, оскільки робити це припущення небезпечно.

```js
// Example Setup
declare var foo:{bar?: {baz: string}};
function immediate(callback: ()=>void) {
  callback();
}


// Type Guard
if (foo.bar) {
  console.log(foo.bar.baz); // Okay
  functionDoingSomeStuff(() => {
    console.log(foo.bar.baz); // TS error: Object is possibly 'undefined'"
  });
}
```

Виправити це так само просто, як зберегти виведене безпечне значення в локальній змінній, автоматично гарантуючи, що воно не буде змінено ззовні, і TypeScript може легко зрозуміти, що:

```js
// Type Guard
if (foo.bar) {
  console.log(foo.bar.baz); // Okay
  const bar = foo.bar;
  functionDoingSomeStuff(() => {
    console.log(bar.baz); // Okay
  });
}
```
