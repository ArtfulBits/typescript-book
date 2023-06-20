# Exception Handling
Обробка винятків.

JavaScript має клас `Error`, який можна використовувати для винятків. Ви видаєте помилку з ключовим словом `throw`. Ви можете зловити його за допомогою пари блоків `try` / `catch`, наприклад.

```js
try {
  throw new Error('Something bad happened');
}
catch(e) {
  console.log(e);
}
```

## Error Sub Types
Подтипи помилок.

Окрім вбудованого класу `Error`, є кілька додаткових вбудованих класів помилок, які успадковують від `Error`, які може викликати середовище виконання JavaScript:

### RangeError

Створює екземпляр, що представляє помилку, яка виникає, коли числова змінна або параметр виходить за межі допустимого діапазону.

```js
// Виклик з перевіщенною кількістю аргументів
console.log.apply(console, new Array(1000000000)); // RangeError: Недійсна довжина масиву
```

### ReferenceError

Створює екземпляр, що представляє помилку, яка виникає під час використання недійсного посилання. Наприкллад:

```js
'use strict';
console.log(notValidVar); // ReferenceError: notValidVar не визначено
```

### SyntaxError

Створює екземпляр, що представляє синтаксичну помилку, яка виникає під час аналізу коду, який не є дійсним JavaScript.

```js
1***3; // SyntaxError: Несподіваний маркер *
```

### TypeError

Створює екземпляр, що представляє помилку, яка виникає, коли змінна або параметр має недійсний тип.

```js
('1.2').toPrecision(1); // TypeError: '1.2'.toPrecision не є функцією
```

### URIError

Створює екземпляр, що представляє помилку, яка виникає, коли `encodeURI()` або `decodeURI()` передають недійсні параметри.

```js
decodeURI('%'); // URIError: неправильний URI
```

## Always use `Error`

Початківці розробники JavaScript іноді просто кидають необроблені рядки, наприклад.
```js
try {
  throw 'Something bad happened';
}
catch(e) {
  console.log(e);
}
```

*Don't do that*. Фундаментальна перевага об’єктів `Error` полягає в тому, що вони автоматично відстежують, де вони були створені та що іх викликало за допомогою властивості `stack`.

Необроблені рядки призводять до дуже болісного досвіду налагодження та ускладнюють аналіз помилок із журналів.

## You don't have to `throw` an error

Передавати об’єкт `Error` можна. Це традиційно для коду стилю зворотного виклику Node.js, який приймає зворотні виклики з першим аргументом як об’єкт помилки.

```js
function myFunction (callback: (e?: Error)) {
  doSomethingAsync(function () {
    if (somethingWrong) {
      callback(new Error('This is my error'))
    } else {
      callback();
    }
  });
}
```

## Exceptional cases

`Exceptions should be exceptional`(Винятки мають бути винятковими) — загальний вислів у інформатиці. Є кілька причин, чому це також стосується JavaScript (і TypeScript).

### Unclear where it is thrown

Розглянемо такий фрагмент коду:

```js
try {
  const foo = runTask1();
  const bar = runTask2();
}
catch(e) {
  console.log('Error:', e);
}
```

Наступний розробник не може знати, яка функція може викликати помилку. Людина, яка переглядає код, не може знати, не прочитавши код для task1 / task2 та інших функцій, які вони можуть викликати тощо.

### Makes graceful handling hard

Ви можете спробувати зробити це витонченим, чітко зафіксувавши кожну річ, яка може викинути:

```js
try {
  const foo = runTask1();
}
catch(e) {
  console.log('Error:', e);
}
try {
  const bar = runTask2();
}
catch(e) {
  console.log('Error:', e);
}
```

Але тепер, якщо вам потрібно передати матеріали з першого завдання до другого, код стає безладним: (зверніть увагу на мутацію `foo`, що вимагає `let` + явну потребу анотувати це, оскільки це не можна вивести з повернення `runTask1`) :

```ts
let foo: number; // Notice use of `let` and explicit type annotation
try {
  foo = runTask1();
}
catch(e) {
  console.log('Error:', e);
}
try {
  const bar = runTask2(foo);
}
catch(e) {
  console.log('Error:', e);
}
```

### Not well represented in the type system

Розглянемо функцію:

```ts
function validate(value: number) {
  if (value < 0 || value > 100) throw new Error('Invalid value');
}
```

Використання `Error` для таких випадків є поганою ідеєю, оскільки воно не представлено у визначенні типу для функції перевірки (це `(value:number) => void`). Натомість кращим способом створення методу перевірки буде:

```ts
function validate(value: number): {error?: string} {
  if (value < 0 || value > 100) return {error:'Invalid value'};
}
```

А тепер його представлено в системі типів.

> Якщо ви не хочете обробити помилку дуже загальним (простим/всеохоплюючим тощо), не *генеруйте* помилку.
