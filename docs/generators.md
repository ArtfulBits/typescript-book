## Генератори

`function *` - це синтаксис, який використовується для створення *генераторної функції*. Виклик генераторної функції повертає *об'єкт генератора*. Об'єкт генератора просто слідує інтерфейсу [ітератора][iterator] (тобто функції `next`, `return` та `throw`).

Є дві ключові мотивації застосування генераторних функцій:

### Ліниві ітератори

Генераторні функції можуть бути використані для створення лінивих ітераторів, наприклад, наступна функція повертає **нескінченний** список цілих чисел за запитом:

```ts
function* infiniteSequence() {
    var i = 0;
    while(true) {
        yield i++;
    }
}

var iterator = infiniteSequence();
while (true) {
    console.log(iterator.next()); // { value: xxxx, done: false } forever and ever
}
```

Звичайно, якщо ітератор закінчується, ви отримуєте результат `{ done: true }`, як показано нижче:

```ts
function* idMaker(){
  let index = 0;
  while(index < 3)
    yield index++;
}

let gen = idMaker();

console.log(gen.next()); // { value: 0, done: false }
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { done: true }
```

### Зовнішньо контрольоване виконання

Це частина генераторів, яка дійсно захоплює. Вона дозволяє функції призупиняти своє виконання та передавати контроль (долю) виконання решти функції викликачеві.

Генераторна функція не виконується, коли ви її викликаєте. Вона просто створює об'єкт генератора. Розгляньте наступний приклад разом з прикладом виконання:

```ts
function* generator(){
    console.log('Execution started');
    yield 0;
    console.log('Execution resumed');
    yield 1;
    console.log('Execution resumed');
}

var iterator = generator();
console.log('Starting iteration'); // Це виконається до того, як будь-що в тілі функції генератора виконається
console.log(iterator.next()); // { value: 0, done: false }
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

Якщо ви запустите це, ви отримаєте наступний вивід:

```
$ node outside.js
Starting iteration
Execution started
{ value: 0, done: false }
Execution resumed
{ value: 1, done: false }
Execution resumed
{ value: undefined, done: true }
```

* Функція починає виконуватися лише тоді, коли на об'єкт генератора викликається `next`.
* Функція *призупиняється*, як тільки зустрічається оператор `yield`.
* Функція *відновлюється*, коли викликається `next`.

> Отже, виконання генераторної функції можна контролювати об'єктом генератора.

Наша комунікація за допомогою генератора була переважно односторонньою, де генератор повертав значення для ітератора. Одним надзвичайно потужним функціоналом генераторів у JavaScript є те, що вони дозволяють двосторонню комунікацію (з обмеженнями).

* ви можете контролювати результат `yield` виразу, використовуючи `iterator.next(valueToInject)`
* ви можете викинути виняток на місці виразу `yield`, використовуючи `iterator.throw(error)`

Наступний приклад демонструє `iterator.next(valueToInject)`:

```ts
function* generator() {
    const bar = yield 'foo'; // bar може бути *будь-якого* типу
    console.log(bar); // bar!
}

const iterator = generator();
// Початок виконання до отримання першого значення yield
const foo = iterator.next();
console.log(foo.value); // foo
// Відновлення виконання з внесенням bar
const nextThing = iterator.next('bar');
```

Оскільки `yield` повертає параметр, переданий функції `next` ітератора, а всі функції `next` ітераторів приймають параметр будь-якого типу, TypeScript завжди призначає тип `any` результату оператора `yield` (`bar` вище).

> Вам потрібно самостійно перетворити результат на тип, який ви очікуєте, і переконатися, що до `next` передаються лише значення цього типу (наприклад, шляхом створення додаткового рівня забезпечення типів, який викликає `next` за вас). Якщо сильна типізація має для вас важливе значення, ви можете уникнути двосторонньої комунікації взагалі, а також пакетів, які сильно на неї покладаються (наприклад, redux-saga).

Наступний приклад демонструє `iterator.throw(error)`:

```ts
function* generator() {
    try {
        yield 'foo';
    }
    catch(err) {
        console.log(err.message); // bar!
    }
}

var iterator = generator();
// Початок виконання до отримання першого значення yield
var foo = iterator.next();
console.log(foo.value); // foo
// Відновлення виконання з викиданням винятку 'bar'
var nextThing = iterator.throw(new Error('bar'));
```

Отже, ось підсумок:
* `yield` дозволяє генераторній функції призупиняти свою комунікацію та передавати контроль зовнішній системі
* зовнішня система може вставити значення в тіло генераторної функції
* зовнішня система може викинути виняток в тіло генераторної функції

Як це корисно? Перейдіть до наступного розділу [**async/await**][async-await] та дізнайтеся.

[iterator]:./iterators.md
[async-await]:./async-await.md