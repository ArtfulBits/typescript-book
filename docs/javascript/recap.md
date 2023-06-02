# Ваш JavaScript – це TypeScript

Було (і надалі буде) багато конкурентів між різними компіляторами, які перетворюють іншу *синтаксичну форму* на *JavaScript*. У порівнянні з цими конкурентами, TypeScript відрізняється тим, що ви можете використовувати JavaScript-код безпосередньо у TypeScript. Ви не повинні переписувати весь код, ви можете використовувати існуючий JavaScript як частину свого TypeScript-проекту. TypeScript є розширенням JavaScript і надає додаткову функціональність та можливості, але ви можете використовувати його як підмножину JavaScript. Ось діаграма:

![JavaScript – це TypeScript](https://raw.githubusercontent.com/basarat/typescript-book/master/images/venn.png)

Однак це означає, що вам потрібно вивчити JavaScript (хороша новина в тому, що вам потрібно вивчити *лише* JavaScript). TypeScript просто стандартизує всі способи надання *документації* для JavaScript.

* Просте надання нового синтаксису не допомагає виявляти помилки, але може допомогти писати більш чистий код з меншою кількістю помилок (наприклад, CoffeeScript).
* Створення нової мови віддаляє вас від вашого середовища виконання та спільнот, але може сприяти легшому впровадженню, якщо це вже знайомий стиль програмування (наприклад, Dart - ближчий до розробників Java / C#).

TypeScript - це просто JavaScript з документацією.

> JSNext є відкритим для інтерпретації. Насправді не все, що пропонується для наступної версії JS, потрапляє до браузерів. TypeScript додає підтримку пропозицій лише тоді, коли вони досягають [стадії 3](https://tc39.es/process-document/).

## Покращення JavaScript

TypeScript спробує захистити вас від частин JavaScript, які ніколи не працювали (тому вам не потрібно пам’ятати ці речі):

```ts
[] + []; // JavaScript will give you "" (which makes little sense), TypeScript will error

//
// other things that are nonsensical in JavaScript
// - don't give a runtime error (making debugging hard)
// - but TypeScript will give a compile time error (making debugging unnecessary)
//
{} + []; // JS : 0, TS Error
[] + {}; // JS : "[object Object]", TS Error
{} + {}; // JS : NaN or [object Object][object Object] depending upon browser, TS Error
"hello" - 1; // JS : NaN, TS Error

function add(a,b) {
  return
    a + b; // JS : undefined, TS Error 'unreachable code detected'
}
```

По суті, TypeScript аналізує JavaScript, подібно до інших лінтерів, але робить це краще за рахунок наявності *інформації про типи*.

## Ви все одно повинні вивчити JavaScript.

Це правда, TypeScript дуже прагматично відноситься до факту, що ви все ж таки *використовуєте JavaScript*, тому є кілька речей про JavaScript, які вам все одно потрібно знати, щоб не потрапити в незручне положення. Давайте обговоримо їх далі.

> Примітка: TypeScript - це підмножина JavaScript. Тільки з документацією, яка може бути використана компіляторами / IDE ;)
