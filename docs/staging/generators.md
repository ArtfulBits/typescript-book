### Generators
Генератори, які також називаються `функцією *`, дозволяють створювати функції, виконання яких можна призупинити, а потім відновити, зберігаючи стан між переходами пауза-відновлення. Значення, яке повертає генератор, називається «ітератором», і його можна використовувати для керування цим переходом «пауза-відновлення».

Ось простий приклад функції генератора, яка генерує *нескінченний* список цілих чисел.

```ts
function* wholeNumbers() {
    var current = 0;
    while(true) {
        yield current++;
    }
}
```

Контекстне ключове слово `yield` використовується для повернення керування від генератора (ефективно призупиняючи виконання функції) разом із необов’язковим значенням (тут `current`). Ви можете отримати доступ до цього значення за допомогою функції-члена `iterator.next()`, це показано нижче:

```ts
function* wholeNumbers() {
    var current = 0;
    while(true) {
        yield current++;
    }
}
var iterator = wholeNumbers();
console.log(iterator.next()); // 0
console.log(iterator.next()); // 1
console.log(iterator.next()); // 2
// so on till infinity....
```

Тепер, коли ви бачили `function*`, `yield` і `.next()`, ми можемо копати глибше.

#### Catching Errors
Будь-які помилки, викинуті генератором (навмисно за допомогою `throw` або випадково через помилку), можуть бути перехоплени за допомогою `try/catch` так само, як і звичайне виконання функції. Це показано нижче:

```ts
function* wholeNumbers() {
    var current = 0;
    while(true) {
      if (current === 3)
        throw new Error('3 is the magic number');
      else
        yield current++;
    }
}
var iterator = wholeNumbers();
console.log(iterator.next()); // 0
console.log(iterator.next()); // 1
console.log(iterator.next()); // 2
try {
    console.log(iterator.next()); // Will throw an error
}
catch(ex) {
    console.log(ex.message); // 3 is the magic number
}
```

#### Controlling function execution externally
Ітератор, повернутий функцією генератора, також можна використовувати для керування станом *всередині* функції генератора.

// TODO: приклад
