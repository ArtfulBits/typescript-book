## Async Await

> [Відеокурс, який охоплює той самий матеріал](https://egghead.io/courses/async-await-using-typescript)

Як експеримент уявіть наступне: спосіб повідомити середовищу виконання JavaScript призупинити виконання коду за ключовим словом await , коли воно використовується для проміса (promise), і відновити лише один раз (і якщо) promise, який повернула функція, був виконан:

```ts
// демо-код. Виключно для експеріменту
async function foo() {
    try {
        var val = await getMeAPromise();
        console.log(val);
    }
    catch(err) {
        console.log('Error: ', err.message);
    }
}
```

Коли проміс завершено, виконання програми продовжується,
* якщо він був виконаний вдало, то await поверне значення,
* якщо проміс буде відхилено, синхронно буде видано помилку, яку ми зможемо зловити.

Це раптово (і чарівним чином) робить асинхронне програмування таким же простим, як і синхронне. Для цього експерименту необхідні три речі:

* Можливість призупинити виконання функції .
* Можливість помістити значення всередину функції.
* Можливість створити виняток усередині функції.

Це саме те, що нам дозволили генератори! Штучний експеримент насправді реальний , як і реалізація async / await у TypeScript / JavaScript. Під кришкою він просто використовує генератори.

### Згенерований JavaScript

Вам не обов’язково це розуміти, але це досить просто, якщо ви читали про генератори . Функцію foo можна просто скласти так:
```ts
const foo = wrapToReturnPromise(function* () {
    try {
        var val = yield getMeAPromise();
        console.log(val);
    }
    catch(err) {
        console.log('Error: ', err.message);
    }
});
```

де wrapToReturnPromise просто виконує функцію генератора, щоб отримати генератор , а потім використовує generator.next(). Якщо значенням є проміс , то він буде мати методи `then`+`catch` та, залежно від результату, викликає generator.next(result) або generator.throw (помилка) . Це воно!


### Підтримка Async Await у TypeScript
**Async - Await** підтримується [TypeScript since version 1.7](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-7.html). Асинхронні функції мають префікс ключового слова  *async*; *await* призупиняє виконання, доки не буде повернуто проміс з асинхронної функції, і розгортає значення з повернутого *Promise*.
Він підтримувався лише для **target es6** , який перетворюється безпосередньо до **ES6 generators**.

**TypeScript 2.1** [додано до ES3 and ES5 run-times](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html), тобто ви можете вільно користуватися ним незалежно від середовища, яке ви використовуєте. Важливо зауважити, що ми можемо використовувати async / await з TypeScript 2.1 і багато браузерів підтримують його при глобальном додаванні **polyfill for Promise**.

Давайте переглянемо цей **приклад** і цей код, щоб зрозуміти, як працює async / await **нотація**: 
```ts
function delay(milliseconds: number, count: number): Promise<number> {
    return new Promise<number>(resolve => {
            setTimeout(() => {
                resolve(count);
            }, milliseconds);
        });
}

// async function always returns a Promise
async function dramaticWelcome(): Promise<void> {
    console.log("Hello");

    for (let i = 0; i < 5; i++) {
        // await is converting Promise<number> into number
        const count: number = await delay(500, i);
        console.log(count);
    }

    console.log("World!");
}

dramaticWelcome();
```

**Transpiling to ES6 (--target es6)**
```js
var __awaiter = (this && this.__awaiter) || function (thisArg, _arguments, P, generator) {
    return new (P || (P = Promise))(function (resolve, reject) {
        function fulfilled(value) { try { step(generator.next(value)); } catch (e) { reject(e); } }
        function rejected(value) { try { step(generator["throw"](value)); } catch (e) { reject(e); } }
        function step(result) { result.done ? resolve(result.value) : new P(function (resolve) { resolve(result.value); }).then(fulfilled, rejected); }
        step((generator = generator.apply(thisArg, _arguments || [])).next());
    });
};
function delay(milliseconds, count) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(count);
        }, milliseconds);
    });
}
// async function always returns a Promise
function dramaticWelcome() {
    return __awaiter(this, void 0, void 0, function* () {
        console.log("Hello");
        for (let i = 0; i < 5; i++) {
            // await is converting Promise<number> into number
            const count = yield delay(500, i);
            console.log(count);
        }
        console.log("World!");
    });
}
dramaticWelcome();
```
You can see full example [here][asyncawaites6code].


**Transpiling to ES5 (--target es5)**
```js
var __awaiter = (this && this.__awaiter) || function (thisArg, _arguments, P, generator) {
    return new (P || (P = Promise))(function (resolve, reject) {
        function fulfilled(value) { try { step(generator.next(value)); } catch (e) { reject(e); } }
        function rejected(value) { try { step(generator["throw"](value)); } catch (e) { reject(e); } }
        function step(result) { result.done ? resolve(result.value) : new P(function (resolve) { resolve(result.value); }).then(fulfilled, rejected); }
        step((generator = generator.apply(thisArg, _arguments || [])).next());
    });
};
var __generator = (this && this.__generator) || function (thisArg, body) {
    var _ = { label: 0, sent: function() { if (t[0] & 1) throw t[1]; return t[1]; }, trys: [], ops: [] }, f, y, t, g;
    return g = { next: verb(0), "throw": verb(1), "return": verb(2) }, typeof Symbol === "function" && (g[Symbol.iterator] = function() { return this; }), g;
    function verb(n) { return function (v) { return step([n, v]); }; }
    function step(op) {
        if (f) throw new TypeError("Generator is already executing.");
        while (_) try {
            if (f = 1, y && (t = y[op[0] & 2 ? "return" : op[0] ? "throw" : "next"]) && !(t = t.call(y, op[1])).done) return t;
            if (y = 0, t) op = [0, t.value];
            switch (op[0]) {
                case 0: case 1: t = op; break;
                case 4: _.label++; return { value: op[1], done: false };
                case 5: _.label++; y = op[1]; op = [0]; continue;
                case 7: op = _.ops.pop(); _.trys.pop(); continue;
                default:
                    if (!(t = _.trys, t = t.length > 0 && t[t.length - 1]) && (op[0] === 6 || op[0] === 2)) { _ = 0; continue; }
                    if (op[0] === 3 && (!t || (op[1] > t[0] && op[1] < t[3]))) { _.label = op[1]; break; }
                    if (op[0] === 6 && _.label < t[1]) { _.label = t[1]; t = op; break; }
                    if (t && _.label < t[2]) { _.label = t[2]; _.ops.push(op); break; }
                    if (t[2]) _.ops.pop();
                    _.trys.pop(); continue;
            }
            op = body.call(thisArg, _);
        } catch (e) { op = [6, e]; y = 0; } finally { f = t = 0; }
        if (op[0] & 5) throw op[1]; return { value: op[0] ? op[1] : void 0, done: true };
    }
};
function delay(milliseconds, count) {
    return new Promise(function (resolve) {
        setTimeout(function () {
            resolve(count);
        }, milliseconds);
    });
}
// async function always returns a Promise
function dramaticWelcome() {
    return __awaiter(this, void 0, void 0, function () {
        var i, count;
        return __generator(this, function (_a) {
            switch (_a.label) {
                case 0:
                    console.log("Hello");
                    i = 0;
                    _a.label = 1;
                case 1:
                    if (!(i < 5)) return [3 /*break*/, 4];
                    return [4 /*yield*/, delay(500, i)];
                case 2:
                    count = _a.sent();
                    console.log(count);
                    _a.label = 3;
                case 3:
                    i++;
                    return [3 /*break*/, 1];
                case 4:
                    console.log("World!");
                    return [2 /*return*/];
            }
        });
    });
}
dramaticWelcome();
```
Ви можете переглянути повний приклад [тут][asyncawaites5code].


**Note**: для обох цільових сценаріїв нам потрібно переконатися, що наше середовище виконання має ECMAScript-сумісний с Promise. Цього можно дістатися при вживанні поліфілу для Promise. Нам також потрібно переконатися, що TypeScript знає про існування Promise, встановивши бібліотеку на кшталт "dom", "es2015" or "dom", "es2015.promise", "es5". 
**Ми можемо побачити, які браузери ДІЙСНО мають підтримку Promise (нативну та з використанням поліфілу) [тут](https://kangax.github.io/compat-table/es6/#test-Promise).**

[generators]:./generators.md
[asyncawaites5code]:https://cdn.rawgit.com/basarat/typescript-book/705e4496/code/async-await/es5/asyncAwaitES5.js
[asyncawaites6code]:https://cdn.rawgit.com/basarat/typescript-book/705e4496/code/async-await/es6/asyncAwaitES6.js
