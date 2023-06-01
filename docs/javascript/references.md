## Посилання (references)

Окрім літералів, будь-який об'єкт у JavaScript (включаючи функції, масиви, regexp тощо) є посиланням. Це означає наступне

### Мутації є у всіх посилання

```js
var foo = {};
var bar = foo; // bar is a reference to the same object

foo.baz = 123;
console.log(bar.baz); // 123
```

### Порівнення при посиланнях
У JavaScript, при порівнянні об'єктів або масивів оператор рівності (== або ===) порівнює посилання, а не фактичний зміст об'єктів. Це означає, що два об'єкти або масиви вважаються рівними лише тоді, коли вони посилаються на ту саму область пам'яті.

```js
var foo = {};
var bar = foo; // bar is a reference
var baz = {}; // baz is a *new object* distinct from `foo`

console.log(foo === bar); // true
console.log(foo === baz); // false
```
