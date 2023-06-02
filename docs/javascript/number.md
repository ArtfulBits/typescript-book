## Число
Кожного разу, коли ви працюєте з числами в будь-якій мові програмування, ви повинні знати про особливості роботи з числами в цій мові. Ось кілька важливих фрагментів інформації про числа в JavaScript, про які вам варто знати.

### Основний тип
У JavaScript є лише один тип чисел. Це 64-бітне число подвійної точності `Number`. Нижче ми обговоримо його обмеження разом з рекомендованим рішенням.

### Десятковий тип
Для тих, хто знайомий з типами даних "double" або "float" в інших мовах програмування, відомо, що двійкові плаваючі числа *не відображаються правильно* на десяткові числа. На приведеному нижче прикладі з використанням вбудованих чисел JavaScript це можна побачити:

```js
console.log(.1 + .2); // 0.30000000000000004
```

> Для справжньої десяткової математики використовуйте `big.js`, згаданий нижче.

### Цілі числа
Максимальне та мінімальне значення цілих чисел, представлене вбудованим типом `Number`, це `Number.MAX_SAFE_INTEGER` та `Number.MIN_SAFE_INTEGER`.

```js
console.log({max: Number.MAX_SAFE_INTEGER, min: Number.MIN_SAFE_INTEGER});
// {max: 9007199254740991, min: -9007199254740991}
```

**Безпечне** в цьому контексті означає, що значення *не може бути результатом помилки округлення*.

Небезпечні значення знаходяться на відстані `+1 / -1` від цих безпечних значень, і будь-яке додавання / віднімання призведе до *округлення* результату.

```js
console.log(Number.MAX_SAFE_INTEGER + 1 === Number.MAX_SAFE_INTEGER + 2); // true!
console.log(Number.MIN_SAFE_INTEGER - 1 === Number.MIN_SAFE_INTEGER - 2); // true!

console.log(Number.MAX_SAFE_INTEGER);      // 9007199254740991
console.log(Number.MAX_SAFE_INTEGER + 1);  // 9007199254740992 - Correct
console.log(Number.MAX_SAFE_INTEGER + 2);  // 9007199254740992 - Rounded!
console.log(Number.MAX_SAFE_INTEGER + 3);  // 9007199254740994 - Rounded - correct by luck
console.log(Number.MAX_SAFE_INTEGER + 4);  // 9007199254740996 - Rounded!
```

Для перевірки безпеки можна використовувати ES6 `Number.isSafeInteger`:

```js
// Safe value
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER)); // true

// Unsafe value
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1)); // false

// Because it might have been rounded to it due to overflow
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 10)); // false
```

> JavaScript з часом отримає підтримку [BigInt](https://developers.google.com/web/updates/2018/05/bigint). Наразі, якщо вам потрібна цілочисельна математика довільної точності, використовуйте `big.js`, згаданий нижче.

### big.js
Кожного разу, коли ви використовуєте математику для фінансових розрахунків (наприклад, розрахунок GST, гроші з центами, додавання тощо), використовуйте бібліотеку на кшталт [big.js](https://github.com/MikeMcl/big.js/), яка призначена для
* Точної десяткової математики
* Захисту цілочисельних значень поза межами обмежень

Встановлення просте:
```bash
npm install big.js @types/big.js
```

Приклад швидкого використання:

```js
import { Big } from 'big.js';

export const foo = new Big('111.11111111111111111111');
export const bar = foo.plus(new Big('0.00000000000000000001'));

// To get a number:
const x: number = Number(bar.toString()); // Loses the precision
```

> Не використовуйте цю бібліотеку для математичних обчислень, що застосовуються в інтерфейсах, вимогливих до продуктивності, наприклад, для побудови діаграм, малювання на canvas тощо.

### NaN
Коли деякі обчислення не можуть бути представлені дійсним числом, JavaScript повертає спеціальне значення `NaN`. Класичним прикладом є уявні числа:

```js
console.log(Math.sqrt(-1)); // NaN
```

Зауваження: Перевірка на рівність **не** працює для значень `NaN`. Замість цього використовуйте `Number.isNaN`:

```js
// Don't do this
console.log(NaN === NaN); // false!!

// Do this
console.log(Number.isNaN(NaN)); // true
```

### Нескінченність (Infinity)
Зовнішні межі значень, які можна представити в Number, доступні у вигляді статичних значень `Number.MAX_VALUE` та `-Number.MAX_VALUE`.

```js
console.log(Number.MAX_VALUE);  // 1.7976931348623157e+308
console.log(-Number.MAX_VALUE); // -1.7976931348623157e+308
```

Значення, які виходять за межі діапазону, де точність не змінюється, обмежуються цими межами, наприклад:

```js
console.log(Number.MAX_VALUE + 1 == Number.MAX_VALUE);   // true!
console.log(-Number.MAX_VALUE - 1 == -Number.MAX_VALUE); // true!
```

Значення, які виходять за межі діапазону, де точність змінюється, перетворюються на спеціальні значення `Infinity`/`-Infinity`, наприклад:

```js
console.log(Number.MAX_VALUE + 1e292);  // Infinity
console.log(-Number.MAX_VALUE - 1e292); // -Infinity
```

Звичайно, ці спеціальні значення "Infinity" також виникають при арифметичних операціях, що вимагають їх використання, наприклад:

```js
console.log( 1 / 0); // Infinity
console.log(-1 / 0); // -Infinity
```

Ви можете використовувати ці значення `Infinity` вручну або використовувати статичні члени класу `Number`, як показано нижче:

```js
console.log(Number.POSITIVE_INFINITY === Infinity);  // true
console.log(Number.NEGATIVE_INFINITY === -Infinity); // true
```

На щастя, оператори порівняння (`<` / `>`) надійно працюють зі значеннями "Infinity":

```js
console.log( Infinity >  1); // true
console.log(-Infinity < -1); // true
```

### Інфінітезимальний (Infinitesimal)

Найменше ненульове значення, яке можна представити в Number, доступне як статичне `Number.MIN_VALUE`.

```js
console.log(Number.MIN_VALUE);  // 5e-324
```

Значення, менші за `MIN_VALUE` ("значення недоповнення"), перетворюються на 0.

```js
console.log(Number.MIN_VALUE / 10);  // 0
```

> Далі за аналогією: Подібно до того, як значення, більші за `Number.MAX_VALUE` обмежуються  до INFINITY, значення, менші за `Number.MIN_VALUE` обмежуються до `0`.
