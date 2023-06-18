# Moving Types

Система типів TypeScript є надзвичайно потужною та дозволяє переміщувати та нарізати типи способами, неможливими в жодній іншій окремій мові.

Це пов’язано з тим, що TypeScript розроблено, щоб дозволити вам бездоганно працювати з *highly dynamic* мовою, такою як JavaScript. Тут ми розглянемо кілька прийомів для переміщення типів у TypeScript.

Основна мотивація для них: ви змінюєте щось одне, а все інше просто оновлюється автоматично, і ви отримуєте приємні помилки, якщо щось збирається зламатися, наприклад добре розроблена система обмежень.

## Copying both the Type + Value

Якщо ви хочете перемістити клас, у вас може виникнути спокуса зробити наступне:

```ts
class Foo { }
var Bar = Foo;
var bar: Bar; // ERROR: cannot find name 'Bar'
```

Це помилка, оскільки `var` скопіював лише `Foo` в простір оголошення *variable* тому ви не можете використовувати `Bar` як анотацію типу. Правильним способом є використання ключового слова `import`. Зверніть увагу, що ви можете використовувати ключове слово `import` таким чином, лише якщо ви використовуєте *namespaces* або *modules* (докладніше про це пізніше):

```ts
namespace importing {
    export class Foo { }
}

import Bar = importing.Foo;
var bar: Bar; // Okay
```

Цей трюк `import` працює лише для речей, які є *both type and variable*.

## Capturing the type of a variable

Ви можете використовувати змінну в анотації типу за допомогою оператора `typeof`. Це дозволяє повідомити компілятору, що одна змінна має той самий тип, що й інша. Ось приклад, щоб продемонструвати це:

```ts
var foo = 123;
var bar: typeof foo; // `bar` має тей самий тип як `foo` (`number`)
bar = 456; // Okay
bar = '789'; // ERROR: Тип `string` не співпадає з `number`
```

## Capturing the type of a class member

Ви можете перейти до будь-якого типу об’єкта, який не допускає значення NULL, щоб отримати тип властивості:

```ts
class Foo {
  foo: number; // деякий член, тип якого ми хочемо захопити
}

let bar: Foo['foo']; // `bar` має тип `number`
```

Крім того, подібно до захоплення типу змінної, ви просто оголошуєте змінну виключно для цілей захоплення типу:

```ts
// Чисто для захоплення типу
declare let _foo: Foo;

// Те саме, що й раніше
let bar: typeof _foo.foo; // `bar` має тип `number`
```

## Capturing the type of magic strings

Багато бібліотек і фреймворків JavaScript працюють на необроблених рядках JavaScript. Ви можете використовувати змінні `const`, щоб зафіксувати їх тип, наприклад.

```ts
// Зберіть *тип* і *значення* магічного рядка:
const foo = "Hello World";

// Использование типа:
let bar: typeof foo;

// bar можна призначати лише для `Hello World`

bar = "Hello World"; // Okay!
bar = "anything else "; // Error!
```

У цьому прикладі `bar` має буквальний тип `"Hello World"`. Ми докладніше розглядаємо це в [literal type section](./literal-types.md).

## Capturing Key Names

Оператор `keyof` дозволяє фіксувати назви ключів типу. наприклад ви можете використовувати його для захоплення імен ключів змінної, спершу захопивши її тип за допомогою `typeof`:

```ts
const colors = {
  red: 'reddish',
  blue: 'bluish'
}
type Colors = keyof typeof colors;

let color: Colors; // теж саме let color: "red" | "blue"
color = 'red'; // okay
color = 'blue'; // okay
color = 'anythingElse'; // Error: тип '"anythingElse"' не співпадає з '"red" | "blue"'
```

Це дозволяє досить легко мати такі речі, як рядок enum + константи, як ви щойно бачили в прикладі вище.
