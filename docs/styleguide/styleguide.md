# TypeScript Style Guide and Coding Conventions

> Неофіційний посібник зі стилю TypeScript

Люди запитували мене про мою думку щодо цього. Особисто я не дуже наполягаю на дотриманні цього в своїх командах і проектах, але це допомагає згадувати про це як тай-брейк, коли хтось відчуває потребу мати таку послідовність. Є інші речі, щодо яких я відчуваю набагато більше, і вони описані в [розділі про поради](../tips/main.md) (наприклад, твердження типу погане, налаштування властивостей погане) 🌹.

Ключові розділи:

* [Variable](#variable-and-function)
* [Class](#class)
* [Interface](#interface)
* [Type](#type)
* [Namespace](#namespace)
* [Enum](#enum)
* [`null` vs. `undefined`](#null-vs-undefined)
* [Formatting](#formatting)
* [Single vs. Double Quotes](#quotes)
* [Tabs vs. Spaces](#spaces)
* [Use semicolons](#semicolons)
* [Annotate Arrays as `Type[]`](#array)
* [File Names](#filename)
* [`type` vs `interface`](#type-vs-interface)
* [`==` or `===`](#-or-)

## Variable and Function
* Використовуйте `camelCase` для імен змінних і функцій

> Причина: звичайний JavaScript

**Bad**
```ts
var FooVar;
function BarFunc() { }
```
**Good**
```ts
var fooVar;
function barFunc() { }
```

## Class
* Використовуйте `PascalCase` для імен класів.

> Причина: це насправді досить традиційно для стандартного JavaScript.

**Bad**
```ts
class foo { }
```
**Good**
```ts
class Foo { }
```
* Використовуйте `camelCase` членів класу та методів

> Причина: природно випливає з умов іменування змінних і функцій.

**Bad**
```ts
class Foo {
    Bar: number;
    Baz() { }
}
```
**Good**
```ts
class Foo {
    bar: number;
    baz() { }
}
```
## Interface

* Використовуйте `PascalCase` для імені.

> Причина: Подібно до класу

* Використовуйте `camelCase` для членів.

> Причина: Подібно до класу

* **Не** вживайте префікс `I`

> Причина: нетрадиційне. `lib.d.ts` визначає важливі інтерфейси без `I` (наприклад, Window, Document тощо).

**Bad**
```ts
interface IFoo {
}
```
**Good**
```ts
interface Foo {
}
```

## Type

* Використовуйте `PascalCase` для імені.

> Причина: Подібно до класу

* Використовуйте `camelCase` для учасників.

> Причина: Подібно до класу


## Namespace

* Використовуйте `PascalCase` для імен

> Причина: Конвенція, яку дотримується команда TypeScript. Простори імен фактично є просто класом зі статичними членами. Імена класів: `PascalCase` => Імена простору імен: `PascalCase`

**Bad**
```ts
namespace foo {
}
```
**Good**
```ts
namespace Foo {
}
```

## Enum

* Використовуйте `PascalCase` для імен enum

> Причина: Подібно до класу. Є типом.

**Bad**
```ts
enum color {
}
```
**Good**
```ts
enum Color {
}
```

* Використовуйте `PascalCase` для члена enum

> Причина: Конвенція, якої дотримується команда TypeScript, тобто розробники мови, наприклад `SyntaxKind.StringLiteral`. Також допомагає з перекладом (генерацією коду) інших мов на TypeScript.

**Bad**
```ts
enum Color {
    red
}
```
**Good**
```ts
enum Color {
    Red
}
```

## Null vs. Undefined

* Бажано не використовувати жоден із них у разі явної недоступності

> Причина: ці значення зазвичай використовуються для підтримки узгодженої структури між значеннями. У TypeScript ви використовуєте *types* для позначення структури

**Bad**
```ts
let foo = { x: 123, y: undefined };
```
**Good**
```ts
let foo: { x: number, y?: number } = { x:123 };
```

* Загалом використовуйте `undefined` (замість цього подумайте про повернення об’єкта на зразок `{valid:boolean, value?:Foo}`)

**Bad**
```ts
return null;
```
**Good**
```ts
return undefined;
```

* Використовуйте `null`, якщо це частина API або домовленість

> Причина: це традиційно в Node.js, напр. `error` має значення `null` для зворотних викликів у стилі NodeBack.

**Bad**
```ts
cb(undefined)
```
**Good**
```ts
cb(null)
```

* Використовуйте *правдиву* перевірку для того, для **об’єктів**  `null` or `undefined`

**Bad**
```ts
if (error === null)
```
**Good**
```ts
if (error)
```

* Використовуйте `== null` / `!= null` (а не `===` / `!==`), щоб перевірити наявність `null` / `undefined` на примітивах, оскільки це працює як для `null`/`undefined`, але не інші помилкові значення (наприклад, `''`, `0`, `false`), напр.


**Bad**
```ts
if (error !== null) // does not rule out undefined
```
**Good**
```ts
if (error != null) // rules out both null and undefined
```

## Formatting
Компілятор TypeScript постачається з дуже гарною мовною службою форматування. Незалежно від результату, який він дає за замовчуванням, достатньо, щоб зменшити когнітивне перевантаження команди.

Використовуйте [`tsfmt`](https://github.com/vvakame/typescript-formatter), щоб автоматично форматувати свій код у командному рядку. Крім того, ваша IDE (atom/vscode/vs/sublime) уже має вбудовану підтримку форматування.

приклади:
```ts
// Space before type i.e. foo:<space>string
const foo: string = "hello";
```

## Quotes

* Надавати перевагу одинарним лапкам (`'`), якщо не використовувати екранування.

> Причина: це робить більше команд JavaScript (наприклад, [airbnb](https://github.com/airbnb/javascript), [стандарт](https://github.com/feross/standard), [npm](https: //github.com/npm/npm), [вузол](https://github.com/nodejs/node), [google/angular](https://github.com/angular/angular/), [facebook /react](https://github.com/facebook/react)). Це легше друкувати (на більшості клавіатур не потрібен зсув). [Команда Prettier також рекомендує одинарні лапки](https://github.com/prettier/prettier/issues/1105)

> Подвійні лапки не без переваг: дозволяють легше копіювати вставляти об’єкти в JSON. Дозволяє людям використовувати інші мови для роботи без зміни символу цитати. Дозволяє використовувати апостроф, напр. `He's not going.`. Але я не хочу відхилятися від того, що спільнота JS вирішила справедливо.

* Якщо ви не можете використовувати подвійні лапки, спробуйте використовувати зворотні галочки (\`).

> Причина: вони зазвичай представляють намір досить складних рядків.

## Spaces

* Використовуйте два пробіли, не табуляцію.

> Причина: це робить більше команд JavaScript (наприклад, [airbnb](https://github.com/airbnb/javascript), [idiomatic](https://github.com/rwaldron/idiomatic.js), [standard]( https://github.com/feross/standard), [npm](https://github.com/npm/npm), [вузол](https://github.com/nodejs/node), [google/ angular](https://github.com/angular/angular/), [facebook/react](https://github.com/facebook/react)). Команди TypeScript/VSCode використовують 4 пробіли, але, безперечно, є винятком у екосистемі.

## Semicolons

* Використовуйте крапку з комою.

> Причини: явні крапки з комою допомагають інструментам форматування мови давати послідовні результати. Відсутній ASI (автоматична вставка крапки з комою) може призвести до помилок, напр. `foo() \n (function(){})` буде одним оператором (а не двома). TC39 [попередження про це також](https://github.com/tc39/ecma262/pull/1062). Приклади команд: [airbnb](https://github.com/airbnb/javascript), [idiomatic](https://github.com/rwaldron/idiomatic.js), [google/angular](https://github .com/angular/angular/), [facebook/react](https://github.com/facebook/react), [Microsoft/TypeScript](https://github.com/Microsoft/TypeScript/).

## Array

* Анотуйте масиви як `foos: Foo[]` замість `foos: Array<Foo>`.

> Причини: легше читати. Він використовується командою TypeScript. Полегшує пізнання того, що щось є масивом, оскільки розум навчений виявляти `[]`.

## Filename
Назвіть файли за допомогою `camelCase`. наприклад `utils.ts`, `map.ts` тощо.

> Причина: традиційно для багатьох команд JS.

Коли файл експортує компонент, а ваша структура (як-от React) хоче, щоб компонент був застосований до PascalCased, використовуйте назву файлу регістра pascal, щоб відповідати, наприклад, `Accordion.tsx`, `MyControl.tsx`.

> Причина: допомагає забезпечити послідовність  і те, що робить екосистема.

## type vs. interface

* Використовуйте `type`, коли вам *може* знадобитися об'єднання або intersection:

type Foo = number | { someProperty: number }
```
* Використовуйте `interface`, коли ви хочете `extends` або `implements`, напр.

```
interface Foo {
  foo: string;
}
interface FooBar extends Foo {
  bar: string;
}
class X implements FooBar {
  foo: string;
  bar: string;
}
```
* В іншому випадку використовуйте те, що робить вас щасливими в цей день. Я використовую [type](https://www.youtube.com/watch?v=IXAT3If0pGI)

## `==` або `===`
Обидва [здебільшого безпечні для користувачів TypeScript](https://www.youtube.com/watch?v=vBhRXMDlA18). Я використовую `===`, оскільки це те, що використовується в кодовій базі TypeScript.

