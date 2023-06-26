## Nominal Typing
Система типів TypeScript є структурною [і це одна з головних мотиваційних переваг](../why-typescript.md). Однак існують випадки реального використання системи, де потрібно, щоб дві змінні були розрізнені, оскільки вони мають різні *назви типу*, навіть якщо вони мають однакову структуру. Дуже поширеним випадком використання є структури *ідентичності* (які, як правило, являють собою просто рядки з семантикою, пов’язаною з їх *ім’ям* у таких мовах, як C#/Java).

Є кілька моделей, які виникли в спільноті. Я розглядаю їх у порядку зменшення особистих переваг:

## Using literal types

У цьому шаблоні використовуються узагальнені та літеральні типи:

```ts
/** Generic Id type */
type Id<T extends string> = {
  type: T,
  value: string,
}

/** Specific Id types */
type FooId = Id<'foo'>;
type BarId = Id<'bar'>;

/** Optional: constructors functions */
const createFoo = (value: string): FooId => ({ type: 'foo', value });
const createBar = (value: string): BarId => ({ type: 'bar', value });

let foo = createFoo('sample')
let bar = createBar('sample');

foo = bar; // Error
foo = foo; // Okay
```

* Переваги
   - Немає потреби в твердженнях будь-якого типу
* Недолік
   - Структура `{type,value}` може бути небажаною та потребувати підтримки серверної серіалізації

## Using Enums
[Enums у TypeScript](../enums.md) пропонують певний рівень номінальної типізації. Два типи перерахувань не є рівноправними, якщо вони відрізняються за назвою. Ми можемо використовувати цей факт для забезпечення номінальної типізації для типів, які в іншому структурно сумісні.

Обхідний шлях передбачає:
* Створення переліку *brand*.
* Створення типу як *перетину* (`&`) переліку бренду + фактичної структури.

Це показано нижче, де структура типів є лише рядком:

```ts
// FOO
enum FooIdBrand { _ = "" };
type FooId = FooIdBrand & string;

// BAR
enum BarIdBrand  { _ = "" };
type BarId = BarIdBrand & string;

/**
 * Usage Demo
 */
var fooId: FooId;
var barId: BarId;

// Safety!
fooId = barId; // error
barId = fooId; // error

// Newing up
fooId = 'foo' as FooId;
barId = 'bar' as BarId;

// Both types are compatible with the base
var str: string;
str = fooId;
str = barId;
```

Зверніть увагу, що переліки брендів FooIdBrand і BarIdBrand вище мають один елемент (_), який відповідає порожньому рядку, як зазначено в `_ = ""`. Це змушує TypeScript зробити висновок, що це переліки на основі рядків зі значеннями типу ``string``, а не переліки зі значеннями типу ``number``. Це необхідно, оскільки TypeScript робить висновок, що порожнє перелічення (``{}``) є числовим переліком, а з TypeScript 3.6.2 перетин числового ``enum`` і ``string`` є ``never ``.

## Using Interfaces

Оскільки `numbers` є типом сумісним з `enum`, попередня техніка не може бути використана для них. Замість цього ми можемо використовувати інтерфейси, щоб порушити структурну сумісність. Цей метод досі використовується командою компіляторів TypeScript, тому варто згадати. Використання префікса `_` і суфікса `Brand` є угодою, яку я настійно рекомендую (і [той, якої дотримується команда TypeScript](https://github.com/Microsoft/TypeScript/blob/7b48a182c05ea4dea81bab73ecbbe9e013a79e99/src/compiler/types .ts#L693-L698)).

Обхідний шлях передбачає наступне:
* додавання невикористаної властивості до типу для порушення структурної сумісності.
* використання твердження типу, коли потрібно оновити або ліквідувати.

Це показано нижче:

```ts
// FOO
interface FooId extends String {
    _fooIdBrand: string; // To prevent type errors
}

// BAR
interface BarId extends String {
    _barIdBrand: string; // To prevent type errors
}

/**
 * Usage Demo
 */
var fooId: FooId;
var barId: BarId;

// Safety!
fooId = barId; // error
barId = fooId; // error
fooId = <FooId>barId; // error
barId = <BarId>fooId; // error

// Newing up
fooId = 'foo' as any;
barId = 'bar' as any;

// If you need the base string
var str: string;
str = fooId as any;
str = barId as any;
```
