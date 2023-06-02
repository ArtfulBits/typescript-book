* [Type Guard](#type-guard)
* [Користувацькі визначені типові охоронці](#user-defined-type-guards)

## Type Guard
Типові охоронці дозволяють звузити тип об'єкта в межах умовного блоку. 


### typeof

TypeScript знає про використання операторів JavaScript `instanceof` та `typeof`. Якщо ви використовуєте їх в умовному блоку, TypeScript зрозуміє, що тип змінної відрізняється в межах цього умовного блоку. Ось швидкий приклад, де TypeScript розуміє, що певна функція не існує в `string` та вказує на те, що це, ймовірно, була помилка користувача:

```ts
function doSomething(x: number | string) {
    if (typeof x === 'string') { // В межах блоку TypeScript знає, що `x` має бути рядком
        console.log(x.subtr(1)); // Помилка, 'subtr' не існує в `string`
        console.log(x.substr(1)); // ОК
    }
    x.substr(1); // Помилка: немає гарантії, що `x` є `string`
}
```

### instanceof

Ось приклад з класом та `instanceof`:

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
        console.log(arg.foo); // ОК
        console.log(arg.bar); // Помилка!
    }
    if (arg instanceof Bar) {
        console.log(arg.foo); // Помилка!
        console.log(arg.bar); // ОК
    }

    console.log(arg.common); // ОК
    console.log(arg.foo); // Помилка!
    console.log(arg.bar); // Помилка!
}

doStuff(new Foo());
doStuff(new Bar());
```

TypeScript навіть розуміє `else`, тому коли `if` звужує один тип, він знає, що в межах `else *це точно не той тип*. Ось приклад:

```ts
class Foo {
    foo = 123;
}

class Bar {
    bar = 123;
}

function doStuff(arg: Foo | Bar) {
    if (arg instanceof Foo) {
        console.log(arg.foo); // ОК
        console.log(arg.bar); // Помилка!
    }
    else {  // МУСИТЬ БУТИ Bar!
        console.log(arg.foo); // Помилка!
        console.log(arg.bar); // ОК
    }
}

doStuff(new Foo());
doStuff(new Bar());
```

### in 

Оператор `in` перевіряє наявність властивості на об'єкті та може використовуватися як типовий охоронець. Наприклад: 

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

Ви можете використовувати `===` / `==` / `!==` / `!=` для розрізнення літеральних значень

```ts
type TriState = 'yes' | 'no' | 'unknown';

function logOutState(state:TriState) {
  if (state == 'yes') {
    console.log('Користувач вибрав так');
  } else if (state == 'no') {
    console.log('Користувач вибрав ні');
  } else {
    console.log('Користувач ще не зробив вибір');
  }
}
```

Це працює навіть тоді, коли у злитті є літеральні типи. Ви можете перевірити значення імені спільної властивості, щоб дискримінувати злиття, наприклад: 

```ts
type Foo = {
  kind: 'foo', // Літеральний тип 
  foo: number
}
type Bar = {
  kind: 'bar', // Літеральний тип 
  bar: number
}

function doStuff(arg: Foo | Bar) {
    if (arg.kind === 'foo') {
        console.log(arg.foo); // ОК
        console.log(arg.bar); // Помилка!
    }
    else {  // МУСИТЬ БУТИ Bar!
        console.log(arg.foo); // Помилка!
        console.log(arg.bar); // ОК
    }
}
```

### null та undefined з `strictNullChecks`

TypeScript досить розумний, щоб виключити як `null`, так і `undefined` з перевіркою `== null` / `!= null`. Наприклад:

```ts
function foo(a?: number | null) {
  if (a == null) return;

  // a тепер число.
}
```

### Користувацькі визначені типові охоронці
JavaScript не має дуже багато підтримки рантаймової інтроспекції. Коли ви використовуєте просто об'єкти JavaScript (використовуючи структурне типізування на свою користь), ви навіть не маєте доступу до `instanceof` або `typeof`. У таких випадках ви можете створити *функції користувацького визначення типу охоронців*. Це просто функції, які повертають `someArgumentName is SomeType`. Ось приклад:

```ts
/**
 * Просто деякі інтерфейси
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
 * Користувацький визначений типовий охоронець!
 */
function isFoo(arg: any): arg is Foo {
    return arg.foo !== undefined;
}

/**
 * Приклад використання користувацького визначеного типового охоронця
 */
function doStuff(arg: Foo | Bar) {
    if (isFoo(arg)) {
        console.log(arg.foo); // ОК
        console.log(arg.bar); // Помилка!
    }
    else {
        console.log(arg.foo); // Помилка!
        console.log(arg.bar); // ОК
    }
}

doStuff({ foo: 123, common: '123' });
doStuff({ bar: 123, common: '123' });
```

### Типові охоронці та зворотні виклики

TypeScript не припускає, що типові охоронці залишаються активними в зворотніх викликах, оскільки це припущення є небезпечним. наприклад: 

```js
// Приклад налаштування
declare var foo:{bar?: {baz: string}};
function immediate(callback: ()=>void) {
  callback();
}


// Типовий охоронець
if (foo.bar) {
  console.log(foo.bar.baz); // ОК
  functionDoingSomeStuff(() => {
    console.log(foo.bar.baz); // Помилка TS: Об'єкт може бути 'undefined'"
  });
}
```

Виправлення таке ж просте, як збереження виведеного безпечного значення в локальній змінній, автоматично забезпечуючи, що воно не буде змінене ззовні, і TypeScript легко розуміє це: 

```js
// Типовий охоронець
if (foo.bar) {
  console.log(foo.bar.baz); // ОК
  const bar = foo.bar;
  functionDoingSomeStuff(() => {
    console.log(bar.baz); // ОК
  });
}
```