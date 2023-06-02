## Обіцянка

Клас `Promise` існує в багатьох сучасних двигунах JavaScript і може бути легко [поліфілленим][polyfill]. Основна мотивація для обіцянок полягає в тому, щоб принести синхронний стиль обробки помилок до коду стилю Async / Callback.

### Код стилю Callback

Щоб повністю оцінити обіцянки, давайте наведемо простий приклад, який доводить складність створення надійного асинхронного коду з простими зворотними викликами. Розгляньте простий випадок створення асинхронної версії завантаження JSON з файлу. Синхронна версія цього може бути досить простою:

```ts
import fs = require('fs');

function loadJSONSync(filename: string) {
    return JSON.parse(fs.readFileSync(filename));
}

// good json file
console.log(loadJSONSync('good.json'));

// non-existent file, so fs.readFileSync fails
try {
    console.log(loadJSONSync('absent.json'));
}
catch (err) {
    console.log('absent.json error', err.message);
}

// invalid json file i.e. the file exists but contains invalid JSON so JSON.parse fails
try {
    console.log(loadJSONSync('invalid.json'));
}
catch (err) {
    console.log('invalid.json error', err.message);
}
```

Є три поведінки цієї простої функції `loadJSONSync`: дійсне значення повернення, помилка файлової системи або помилка JSON.parse. Ми обробляємо помилки за допомогою простого try/catch, як ви звикли робити синхронне програмування в інших мовах програмування. Тепер давайте створимо хорошу асинхронну версію такої функції. Прийнятний початковий варіант з тривіальною логікою перевірки помилок буде наступним:

```ts
import fs = require('fs');

// Прийнятний початковий варіант .... але не правильний. Ми пояснимо причини нижче
function loadJSON(filename: string, cb: (error: Error, data: any) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) cb(err);
        else cb(null, JSON.parse(data));
    });
}
```

Досить просто, він приймає зворотний виклик, передає будь-які помилки файлової системи зворотному виклику. Якщо помилок файлової системи немає, він повертає результат `JSON.parse`. Декілька моментів, які потрібно запам'ятати при роботі з асинхронними функціями на основі зворотних викликів:

1. Ніколи не викликайте зворотний виклик двічі.
1. Ніколи не викидайте помилку.

Однак ця проста функція не враховує другий пункт. Фактично, `JSON.parse` викидає помилку, якщо передається поганий JSON, і зворотний виклик ніколи не викликається, і програма аварійно завершується. Це демонструється в нижченаведеному прикладі:

```ts
import fs = require('fs');

// Прийнятний початковий варіант .... але не правильний
function loadJSON(filename: string, cb: (error: Error, data: any) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) cb(err);
        else cb(null, JSON.parse(data));
    });
}

// load invalid json
loadJSON('invalid.json', function (err, data) {
    // Цей код ніколи не виконується
    if (err) console.log('bad.json error', err.message);
    else console.log(data);
});
```

Наївна спроба виправити це полягає в тому, щоб обгорнути `JSON.parse` в try catch, як показано в нижченаведеному прикладі:

```ts
import fs = require('fs');

// Краща спроба ... але все ще не правильна
function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) {
            cb(err);
        }
        else {
            try {
                cb(null, JSON.parse(data));
            }
            catch (err) {
                cb(err);
            }
        }
    });
}

// load invalid json
loadJSON('invalid.json', function (err, data) {
    if (err) console.log('bad.json error', err.message);
    else console.log(data);
});
```

Однак у цьому коді є хитра помилка. Якщо зворотний виклик (`cb`), а не `JSON.parse`, викидає помилку, оскільки ми обгорнули його в `try`/`catch`, виконується `catch`, і ми знову викликаємо зворотний виклик, тобто зворотний виклик викликається двічі! Це демонструється в прикладі нижче:

```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) {
            cb(err);
        }
        else {
            try {
                cb(null, JSON.parse(data));
            }
            catch (err) {
                cb(err);
            }
        }
    });
}

// a good file but a bad callback ... gets called again!
loadJSON('good.json', function (err, data) {
    console.log('our callback called');

    if (err) console.log('Error:', err.message);
    else {
        // let's simulate an error by trying to access a property on an undefined variable
        var foo;
        // The following code throws `Error: Cannot read property 'bar' of undefined`
        console.log(foo.bar);
    }
});
```

```bash
$ node asyncbadcatchdemo.js
our callback called
our callback called
Error: Cannot read property 'bar' of undefined
```

Це тому, що наша функція `loadJSON` неправильно обгорнула зворотний виклик в блок `try`. Тут є простий урок, який потрібно запам'ятати.

> Простий урок: Містіть усі свій синхронний код у блок try catch, крім випадку, коли ви викликаєте зворотний виклик.

Дотримуючись цього простого уроку, ми маємо повністю функціональну асинхронну версію `loadJSON`, яка показана нижче:

```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) return cb(err);
        // Містіть усі свій синхронний код у блок try catch
        try {
            var parsed = JSON.parse(data);
        }
        catch (err) {
            return cb(err);
        }
        // крім випадку, коли ви викликаєте зворотний виклик
        return cb(null, parsed);
    });
}
```
Визнаючи це, не важко слідувати за цим, якщо ви це зробили кілька разів, але, тим не менш, це багато коду для написання просто для хорошої обробки помилок. Тепер давайте розглянемо кращий спосіб вирішення асинхронного JavaScript за допомогою обіцянок.

## Створення обіцянки

Обіцянка може бути або `pending` або `fulfilled` або `rejected`.

![стан та долі обіцянок](https://raw.githubusercontent.com/basarat/typescript-book/master/images/promise%20states%20and%20fates.png)

Давайте розглянемо створення обіцянки. Це просто питання виклику `new` на `Promise` (конструктор обіцянок). Конструктор обіцянок отримує функції `resolve` та `reject` для вирішення стану обіцянки:

```ts
const promise = new Promise((resolve, reject) => {
    // функції resolve / reject контролюють долю обіцянки
});
```

### Підписка на долю обіцянки

Долю обіцянки можна підписатися за допомогою `.then` (якщо вона вирішена) або `.catch` (якщо вона відхилена).

```ts
const promise = new Promise((resolve, reject) => {
    resolve(123);
});
promise.then((res) => {
    console.log('Я викликаюся:', res === 123); // Я викликаюся: true
});
promise.catch((err) => {
    // Це ніколи не викликається
});
```

```ts
const promise = new Promise((resolve, reject) => {
    reject(new Error("Щось сталося"));
});
promise.then((res) => {
    // Це ніколи не викликається
});
promise.catch((err) => {
    console.log('Я викликаюся:', err.message); // Я викликаюся: 'Щось сталося'
});
```

> ПОРАДА: Швидкі обіцянки
* Швидке створення вже вирішеної обіцянки: `Promise.resolve(result)`
* Швидке створення вже відхиленої обіцянки: `Promise.reject(error)`

### Ланцюжковість обіцянок
Ланцюжковість обіцянок **є серцем переваг, які надають обіцянки**. Як тільки у вас є обіцянка, з цього моменту ви використовуєте функцію `then`, щоб створити ланцюжок обіцянок.

* Якщо ви повертаєте обіцянку з будь-якої функції в ланцюжку, `.then` викликається лише після того, як значення вирішено:

```ts
Promise.resolve(123)
    .then((res) => {
        console.log(res); // 123
        return 456;
    })
    .then((res) => {
        console.log(res); // 456
        return Promise.resolve(123); // Зверніть увагу, що ми повертаємо обіцянку
    })
    .then((res) => {
        console.log(res); // 123 : Зверніть увагу, що цей `then` викликається з вирішеним значенням
        return 123;
    })
```

* Ви можете агрегувати обробку помилок будь-якої попередньої частини ланцюжка за допомогою одного `catch`:

```ts
// Створити відхилений обіцянку
Promise.reject(new Error('щось пішло не так'))
    .then((res) => {
        console.log(res); // не викликається
        return 456;
    })
    .then((res) => {
        console.log(res); // не викликається
        return 123;
    })
    .then((res) => {
        console.log(res); // не викликається
        return 123;
    })
    .catch((err) => {
        console.log(err.message); // щось пішло не так
    });
```

* `catch` фактично повертає нову обіцянку (ефективно створюючи новий ланцюжок обіцянок):

```ts
// Створити відхилений обіцянку
Promise.reject(new Error('щось пішло не так'))
    .then((res) => {
        console.log(res); // не викликається
        return 456;
    })
    .catch((err) => {
        console.log(err.message); // щось пішло не так
        return 123;
    })
    .then((res) => {
        console.log(res); // 123
    })
```

* Будь-які синхронні помилки, що виникають у `then` (або `catch`), призводять до невдачі поверненої обіцянки:

```ts
Promise.resolve(123)
    .then((res) => {
        throw new Error('щось пішло не так'); // викинути синхронну помилку
        return 456;
    })
    .then((res) => {
        console.log(res); // ніколи не викликається
        return Promise.resolve(789);
    })
    .catch((err) => {
        console.log(err.message); // щось пішло не так
    })
```

* Тільки відповідний (найближчий хвостовий) `catch` викликається для даної помилки (так як catch починає новий ланцюжок обіцянок).

```ts
Promise.resolve(123)
    .then((res) => {
        throw new Error('щось пішло не так'); // викинути синхронну помилку
        return 456;
    })
    .catch((err) => {
        console.log('перший catch: ' + err.message); // щось пішло не так
        return 123;
    })
    .then((res) => {
        console.log(res); // 123
        return Promise.resolve(789);
    })
    .catch((err) => {
        console.log('другий catch: ' + err.message); // ніколи не викликається
    })
```

* `catch` викликається тільки в разі помилки в попередньому ланцюжку:

```ts
Promise.resolve(123)
    .then((res) => {
        return 456;
    })
    .catch((err) => {
        console.log("ТУТ"); // ніколи не викликається
    })
```

Той факт, що:

* помилки переходять до хвостового `catch` (і пропускають будь-які проміжні виклики `then`) і
* синхронні помилки також ловляться будь-яким хвостовим `catch`.

ефективно надає нам парадигму асинхронного програмування, яка дозволяє кращу обробку помилок, ніж прості зворотні виклики. Докладніше про це нижче.


### TypeScript та обіцянки
Чудовою річчю про TypeScript є те, що він розуміє потік значень через ланцюжок обіцянок:

```ts
Promise.resolve(123)
    .then((res) => {
         // res має тип `number`
         return true;
    })
    .then((res) => {
        // res має тип `boolean`

    });
```

Звичайно, він також розуміє розгортання будь-яких викликів функцій, які можуть повернути обіцянку:

```ts
function iReturnPromiseAfter1Second(): Promise<string> {
    return new Promise((resolve) => {
        setTimeout(() => resolve("Привіт світ!"), 1000);
    });
}

Promise.resolve(123)
    .then((res) => {
        // res має тип `number`
        return iReturnPromiseAfter1Second(); // Ми повертаємо `Promise<string>`
    })
    .then((res) => {
        // res має тип `string`
        console.log(res); // Привіт світ!
    });
```


### Конвертування функції у стилі зворотного виклику для повернення обіцянки

Просто оберніть виклик функції в обіцянку та
- `reject`, якщо виникає помилка,
- `resolve`, якщо все добре.

Наприклад, давайте обернемо `fs.readFile`:

```ts
import fs = require('fs');
function readFileAsync(filename: string): Promise<any> {
    return new Promise((resolve,reject) => {
        fs.readFile(filename,(err,result) => {
            if (err) reject(err);
            else resolve(result);
        });
    });
}
```

Найбільш надійним способом є написання цього власноруч, і це не обов'язково повинно бути таким же детальним, як у попередньому прикладі, наприклад, перетворення `setTimeout` в функцію `delay`, яка повертає обіцянку, є дуже простим:

```ts
const delay = (ms: number) => new Promise(res => setTimeout(res, ms));
```

Зверніть увагу, що в NodeJS є зручна функція, яка робить це магію з функцією в стилі `node => promise` за вас:

```ts
/** Приклад використання */
import fs from 'fs';
import util from 'util';
const readFile = util.promisify(fs.readFile);
```

> Webpack підтримує модуль `util` з коробки, і ви можете використовувати його також у браузері.

Якщо у вас є функція у стилі зворотного виклику Node як *член*, переконайтеся, що ви також `bind` його, щоб впевнитися, що він має правильний `this`:

```ts
const dbGet = util.promisify(db.get).bind(db);
```

### Перегляд прикладу JSON

Тепер давайте переглянемо наш приклад `loadJSON` та перепишемо асинхронну версію, яка використовує обіцянки. Все, що нам потрібно зробити, це прочитати вміст файлу як обіцянку, потім розібрати його як JSON, і ми готові. Це проілюстровано в наведеному нижче прикладі:

```ts
function loadJSONAsync(filename: string): Promise<any> {
    return readFileAsync(filename) // Використовуйте функцію, яку ми щойно написали
                .then(function (res) {
                    return JSON.parse(res);
                });
}
```

Використання (зверніть увагу, наскільки воно схоже на початкову `sync` версію, введену на початку цього розділу 🌹):

```ts
// good json file
loadJSONAsync('good.json')
    .then(function (val) { console.log(val); })
    .catch(function (err) {
        console.log('good.json error', err.message); // ніколи не викликається
    })

// non-existent json file
    .then(function () {
        return loadJSONAsync('absent.json');
    })
    .then(function (val) { console.log(val); }) // ніколи не викликається
    .catch(function (err) {
        console.log('absent.json error', err.message);
    })

// invalid json file
    .then(function () {
        return loadJSONAsync('invalid.json');
    })
    .then(function (val) { console.log(val); }) // ніколи не викликається
    .catch(function (err) {
        console.log('bad.json error', err.message);
    });
```

Причина, чому ця функція була простішою, полягає в тому, що об'єднання "`loadFile`(async) + `JSON.parse` (sync) => `catch`" було зроблено ланцюжком обіцянок. Крім того, зворотний виклик не був викликаний *нами*, а викликав його ланцюжок обіцянок, тому ми не мали можливості зробити помилку, обгорнувши його в `try/catch`.

### Паралельний потік управління
Ми побачили, наскільки тривіальним є виконання послідовної послідовності асинхронних задач з обіцянками. Це просто питання ланцюжка викликів `then`.

Однак ви, можливо, захочете запустити серію асинхронних задач, а потім щось зробити з результатами всіх цих задач. `Promise` надає статичну функцію `Promise.all`, яку ви можете використовувати, щоб чекати на завершення `n` кількості обіцянок. Ви надаєте йому масив з `n` обіцянками, і він дає вам масив з `n` розв'язаними значеннями. Нижче ми показуємо Ланцюжок, а також Паралельний:

```ts
// асинхронна функція для імітації завантаження елемента з якогось сервера
function loadItem(id: number): Promise<{ id: number }> {
    return new Promise((resolve) => {
        console.log('loading item', id);
        setTimeout(() => { // імітує затримку сервера
            resolve({ id: id });
        }, 1000);
    });
}

// Ланцюжковий / Послідовний
let item1, item2;
loadItem(1)
    .then((res) => {
        item1 = res;
        return loadItem(2);
    })
    .then((res) => {
        item2 = res;
        console.log('done');
    }); // загальний час буде близько 2 с

// Конкурентний / Паралельний
Promise.all([loadItem(1), loadItem(2)])
    .then((res) => {
        [item1, item2] = res;
        console.log('done');
    }); // загальний час буде близько 1 с
```

Іноді ви хочете запустити серію асинхронних задач, але ви отримуєте все, що вам потрібно, як тільки будь-яка з цих задач вирішена. `Promise` надає статичну функцію `Promise.race` для цього сценарію:

```ts
var task1 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 1000, 'one');
});
var task2 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 2000, 'two');
});

Promise.race([task1, task2]).then(function(value) {
  console.log(value); // "one"
  // Обидва розв'язуються, але task1 розв'язується швидше
});
```

[поліфіл]:https://github.com/stefanpenner/es6-promise