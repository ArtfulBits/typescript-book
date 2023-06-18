# Type Inference in TypeScript

TypeScript може визначити (а потім перевірити) тип змінної на основі кількох простих правил. Тому що ці правила
прості, ви можете навчити свій мозок розпізнавати безпечний/небезпечний код (це сталося зі мною та моїми товаришами по команді досить швидко).

> Потік типів – це саме те, як я уявляю в своєму мозку потік інформації про типи.

## Variable Definition

Типи змінної виводяться за визначенням.
```ts
let foo = 123; // foo is a `number`
let bar = "Hello"; // bar is a `string`
foo = bar; // Error: cannot assign `string` to a `number`
```

Це приклад типів, що перетікають справа наліво.

## Function Return Types

Тип повернення визначається операторами return, наприклад. передбачається, що наступна функція повертає `number`.

```ts
function add(a: number, b: number) {
    return a + b;
}
```

Це приклад типів, що випливають знизу.

## Assignment

Тип параметрів функції/повернених значень також можна визначити за допомогою призначення, наприклад. тут ми говоримо, що `foo` є `Adder`, що робить `number` типом `a` і `b`.

```ts
type Adder = (a: number, b: number) => number;
let foo: Adder = (a, b) => a + b;
```

Цей факт можна продемонструвати наведеним нижче кодом, який викликає помилку, як ви сподіваєтесь:

```ts
type Adder = (a: number, b: number) => number;
let foo: Adder = (a, b) => {
    a = "hello"; // Error: cannot assign `string` to a `number`
    return a + b;
}
```

Це приклад типів, що перетікають зліва направо.

Той самий спосіб визначення типу *призначення* працює, якщо ви створюєте функцію для аргументу зворотного виклику. Зрештою, `argument -> parameter` - це просто інша форма призначення змінних.

```ts
type Adder = (a: number, b: number) => number;
function iTakeAnAdder(adder: Adder) {
    return adder(1, 2);
}
iTakeAnAdder((a, b) => {
    // a = "hello"; // Would Error: cannot assign `string` to a `number`
    return a + b;
})
```

## Structuring

Ці прості правила також працюють за наявності **structuring** (створення буквального об’єкта). Наприклад, у наступному випадку тип `foo` вважається `{a:number, b:number}`
```ts
let foo = {
    a: 123,
    b: 456
};
// foo.a = "hello"; // Would Error: cannot assign `string` to a `number`
```

Аналогічно для масивів:

```ts
const bar = [1,2,3];
// bar[0] = "hello"; // Would error: cannot assign `string` to a `number`
```

І звичайно будь-яке вкладення:

```ts
let foo = {
    bar: [1, 3, 4]
};
// foo.bar[0] = 'hello'; // Would error: cannot assign `string` to a `number`
```

## Destructuring

І, звичайно, вони також працюють з деструктуризацією обох об’єктів:

```ts
let foo = {
    a: 123,
    b: 456
};
let {a} = foo;
// a = "hello"; // Would Error: cannot assign `string` to a `number`
```

and arrays:

```ts
const bar = [1, 2];
let [a, b] = bar;
// a = "hello"; // Would Error: cannot assign `string` to a `number`
```

І якщо параметр функції можна вивести, то можна вивести і його деструктуровані властивості. Наприклад, тут ми деструктуруємо аргумент на члени `a`/`b`.

```ts
type Adder = (numbers: { a: number, b: number }) => number;
function iTakeAnAdder(adder: Adder) {
    return adder({ a: 1, b: 2 });
}
iTakeAnAdder(({a, b}) => { // Types of `a` and `b` are inferred
    // a = "hello"; // Would Error: cannot assign `string` to a `number`
    return a + b;
})
```

## Type Guards

Ми вже бачили, як [Type Guards](./typeGuard.md) допомагає змінювати та звужувати типи (зокрема, у випадку об’єднань). Захисники типу — це просто інша форма виведення типу для змінної в блоці.

## Warnings

### Be careful around parameters

Типи не входять у параметри функції, якщо це не можна вивести з призначення. Наприклад, у наступному випадку компілятор не знає типу `foo`, тому він не може визначити тип `a` або `b`.

```ts
const foo = (a,b) => { /* do something */ };
```

Однак, якщо було введено `foo`, можна визначити тип параметрів функції (обидва `a`, `b` вважаються типом `number` у прикладі нижче).

```ts
type TwoNumberFunction = (a: number, b: number) => void;
const foo: TwoNumberFunction = (a, b) => { /* do something */ };
```

### Be careful around return

Хоча TypeScript загалом може визначити тип повернення функції, це може бути не те, що ви очікуєте. Наприклад, тут функція `foo` має тип повернення `any`.

```ts
function foo(a: number, b: number) {
    return a + addOne(b);
}
// Some external function in a library someone wrote in JavaScript
function addOne(c) {
    return c + 1;
}
```

Це тому, що на тип повернення впливає погане визначення типу для `addOne` (`c` - це `any`, тому повернення `addOne` є `any`, тому повернення `foo` є `any`).

> Я вважаю найпростішим завжди чітко говорити про повернення функції. Зрештою, ці анотації є теоремою, а тіло функції є доказом.

Є й інші випадки, які можна уявити, але хороша новина полягає в тому, що існує прапор компілятора, який може допомогти виявити такі помилки.

## `noImplicitAny`

Прапор `noImplicitAny` в наказує компілятору викликати помилку, якщо він не може визначити тип змінної (і тому може мати її лише як *implicit* `any` тип). Тоді можна

* або скажіть, що *yes I want it to be of type `any`* додавши *explicitly* анотацію типу `: any`
* допоможіть компілятору, додавши ще кілька *вірних* анотацій.
