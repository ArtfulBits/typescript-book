* [lib.d.ts](#libdts)
* [Example Usage](#example-usage)
* [Inside look](#libdts-inside-look)
* [Modifying Native types](#modifying-native-types)
* [Using custom lib.d.ts](#using-your-own-custom-libdts)
* [Compiler `target` effect on lib.d.ts](#compiler-target-effect-on-libdts)
* [`lib` option](#lib-option)
* [Polyfill for old JavaScript engines](#polyfill-for-old-javascript-engines)

## `lib.d.ts`

Спеціальний файл декларації `lib.d.ts` постачається з кожною інсталяцією TypeScript. Цей файл містить оголошення середовища для різних поширених конструкцій JavaScript, присутніх у середовищі виконання JavaScript і DOM.

* Цей файл автоматично включається в контекст компіляції проекту TypeScript.
* Мета цього файлу — полегшити вам початок написання *type checked* коду JavaScript.

Ви можете виключити цей файл із контексту компіляції, вказавши прапор командного рядка компілятора `--noLib` (або `"noLib" : true` у `tsconfig.json`).

### Example Usage

Як завжди, давайте подивимося на приклади використання цього файлу в дії:

```ts
var foo = 123;
var bar = foo.toString();
```
Цей тип коду добре працює, тому що функція `toString` визначена в `lib.d.ts` для всіх об’єктів JavaScript.

Якщо ви використовуєте той самий зразок коду з параметром `noLib`, ви отримаєте помилку перевірки типу:

```ts
var foo = 123;
var bar = foo.toString(); // ERROR: Власність 'toString' не існує для типу 'number'.
```
Тепер, коли ви розумієте важливість `lib.d.ts`, як виглядає його вміст? Ми розглядаємо це далі.

### `lib.d.ts` Inside Look

Вміст `lib.d.ts` — це, насамперед, купа оголошень *variable* , наприклад. `window`, `document`, `math` і купа подібних оголошень *interface* наприклад. `Window` , `Document`, `Math`.

Найпростіший спосіб прочитати документацію та ввести анотації глобальних речей — це ввести код, *tякий 100% працює* наприклад. `Math.floor`, а потім F12 (перейдіть до визначення), використовуючи вашу IDE (VSCode чудово підтримує це).

Давайте подивимося на зразок оголошення *variable* наприклад `window` визначається як:
```ts
declare var window: Window;
```
Це просто `declare var`, за яким слідує назва змінної (тут `window`) та інтерфейс для анотації типу (тут інтерфейс `Window`). Ці змінні зазвичай вказують на якийсь глобальний *interface*, наприклад. ось невеликий зразок (насправді досить великого) інтерфейсу `Window`:

```ts
interface Window extends EventTarget, WindowTimers, WindowSessionStorage, WindowLocalStorage, WindowConsole, GlobalEventHandlers, IDBEnvironment, WindowBase64 {
    animationStartTime: number;
    applicationCache: ApplicationCache;
    clientInformation: Navigator;
    closed: boolean;
    crypto: Crypto;
    // so on and so forth...
}
```
Ви бачите, що в цих інтерфейсах є *багато* інформації про типи. За відсутності TypeScript *вам* потрібно було б тримати це в *вашій* голові. Тепер ви можете завантажити ці знання на компілятор із легким доступом до них за допомогою таких речей, як `intellisense`.

Існує вагома причина для використання *interfaces* для цих глобалів. Це дозволяє вам *додавати додаткові власності* до цих глобалів *без* необхідності змінювати `lib.d.ts`. Далі ми розглянемо це поняття.

### Modifying Native Types

Оскільки `interface` у TypeScript є відкритим, це означає, що ви можете просто додати членів до інтерфейсів, оголошених у `lib.d.ts`, і TypeScript врахує ці доповнення. Зауважте, що вам потрібно внести ці зміни в[*global module*](../project/modules.md) щоб ці інтерфейси були пов’язані з `lib.d.ts`. Ми навіть рекомендуємо створити для цього спеціальний файл під назвою [`global.d.ts`](../project/globals.md).

Ось декілька прикладів випадків, коли ми додаємо матеріал до `window`, `Math`, `Date`:

#### Example `window`

Просто додайте щось до інтерфейсу `Window`, наприклад:

```ts
interface Window {
    helloWorld(): void;
}
```

Це дозволить вам використовувати його *type safe* способом:

```ts
// Додайте його під час виконання
window.helloWorld = () => console.log('hello world');
// викличьте його
window.helloWorld();
// Використовуйте його неправильно, і ви отримаєте помилку:
window.helloWorld('gracius'); // Error: Надані параметри не відповідають сигнатурі цільового виклику
```

#### Example `Math`
Глобальна змінна `Math` визначена в `lib.d.ts` як (знову ж таки, використовуйте інструменти розробника, щоб перейти до визначення):

```ts
/** Внутрішній об’єкт, який надає основні математичні функції та константи. */
declare var Math: Math;
```

тобто змінна `Math` є екземпляром інтерфейсу `Math`. Інтерфейс Math визначається як:

```ts
interface Math {
    E: number;
    LN10: number;
    // others ...
}
```

Це означає, що якщо ви хочете додати щось до глобальної змінної `Math`, вам просто потрібно додати це до глобального інтерфейсу `Math`, наприклад. розгляньте проект [`seedrandom`] (<https://www.npmjs.com/package/seedrandom>), який додає функцію `seedrandom` до глобального об’єкта `Math`. Це можна оголосити досить легко:

```ts
interface Math {
    seedrandom(seed?: string);
}
```

Просто  використайте її:

```ts
Math.seedrandom();
// or
Math.seedrandom("Any string you want!");
```

#### Example `Date`

Якщо ви подивитеся на визначення`Date` *variable* в `lib.d.ts`, ви знайдете:

```ts
declare var Date: DateConstructor;
```
Інтерфейс `DateConstructor` подібний до того, що ви бачили раніше з `Math` і `Window`, оскільки він містить елементи, які ви можете використовувати поза глобальною змінною `Date`, наприклад. `Date.now()`. Окрім цих членів, він містить підписи *construct*, які дозволяють створювати екземпляри `Date` (наприклад, `new Date()`). Фрагмент інтерфейсу `DateConstructor` показано нижче:

```ts
interface DateConstructor {
    new (): Date;
    // ... other construct signatures

    now(): number;
    // ... other member functions
}
```

Розглянемо проект [`datejs`](https://github.com/abritinthebay/datejs). DateJS додає членів як до глобальної змінної `Date`, так і до екземплярів `Date`. Тому визначення TypeScript для цієї бібліотеки виглядатиме так ([BTW the community has already written this for you in this case](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/datejs/index.d.ts)):

```ts
/** DateJS Public Static Methods */
interface DateConstructor {
    /** Отримує дату, для якої встановлено поточну дату. Час встановлюється на початок дня (00:00 або 12:00). */
    today(): Date;
    // ... so on and so forth
}

/** DateJS Public Instance Methods */
interface Date {
    /** Додає вказану кількість мілісекунд до цього екземпляра. */
    addMilliseconds(milliseconds: number): Date;
    // ... so on and so forth
}
```
Це дозволяє вам робити щось на кшталт наступного у безпечний спосіб:

```ts
var today = Date.today();
var todayAfter1second = today.addMilliseconds(1000);
```

#### Example `string`

Якщо ви заглянете всередину `lib.d.ts` для рядка, ви знайдете речі, подібні до того, що ми бачили для `Date` (глобальна змінна `String`, інтерфейс `StringConstructor`, інтерфейс `String`). Однак варто зауважити, що інтерфейс `String` також впливає на рядкові *literals*, як показано в прикладі коду нижче:

```ts

interface String {
    endsWith(suffix: string): boolean;
}

String.prototype.endsWith = function(suffix: string): boolean {
    var str: string = this;
    return str && str.indexOf(suffix, str.length - suffix.length) !== -1;
}

console.log('foo bar'.endsWith('bas')); // false
console.log('foo bas'.endsWith('bas')); // true
```

Подібні змінні та інтерфейси існують для інших речей, які мають як статичні, так і екземпляри, як-от `Number`, `Boolean`, `RegExp` тощо, і ці інтерфейси також впливають на екземпляри літеральних типів цих типів.

### Example `string` redux

Ми рекомендуємо створити `global.d.ts` з причин зручності обслуговування. Однак ви можете проникнути в *global namespace* з *a file module* якщо хочете. Це робиться за допомогою `declare global { /*global namespace here*/ }`. наприклад попередній приклад також можна зробити так:

```ts
// Ensure this is treated as a module.
export {};

declare global {
    interface String {
        endsWith(suffix: string): boolean;
    }
}

String.prototype.endsWith = function(suffix: string): boolean {
    var str: string = this;
    return str && str.indexOf(suffix, str.length - suffix.length) !== -1;
}

console.log('foo bar'.endsWith('bas')); // false
console.log('foo bas'.endsWith('bas')); // true
```

### Using your own custom lib.d.ts
Як ми вже згадували раніше, використання булевого прапорця компілятора `--noLib` змушує TypeScript виключати автоматичне включення `lib.d.ts`. Існують різні причини, чому ця функція корисна. Ось кілька поширених:

* Ви працюєте в спеціальному середовищі JavaScript, яке відрізняється від стандартного середовища виконання на основі браузера.
* Вам подобається мати *strict* контроль над *globals* параметрами, доступними у вашому коді. наприклад lib.d.ts визначає `item` як глобальну змінну, і ви не хочете, щоб це просочувалося у ваш код.

Після того, як ви виключили типовий `lib.d.ts`, ви можете включити файл із подібною назвою до вашого контексту компіляції, і TypeScript підбере його для перевірки типу.

> Примітка: будьте обережні з `--noLib`. Коли ви перебуваєте на території noLib, якщо ви вирішите поділитися своїм проектом з іншими, вони будуть *forced* перейти на noLib (точніше *your lib*). Навіть гірше, якщо ви внесете *their* код у свій проект, вам може знадобитися перенести його на *your lib*.

### Compiler target effect on `lib.d.ts`

Встановлення мети компілятора на `es6` призводить до того, що `lib.d.ts` включає *additional* оголошення середовища для більш сучасних (es6) речей, таких як `Promise`. Цей магічний ефект зміни мети компілятора *ambience* коду є бажаним для одних людей, а для інших це проблематично, оскільки він поєднує *code generation* з *code ambience*.

Однак, якщо ви бажаєте більш детального контролю над вашим середовищем, вам слід скористатися параметром `--lib`, який ми обговоримо далі.

### lib option

Іноді (багато разів) ви хочете роз’єднати зв’язок між метою компіляції (згенерованою версією JavaScript) і підтримкою зовнішньої бібліотеки. Поширеним прикладом є `Promise`, напр. сьогодні (у червні 2016 року) ви, швидше за все, захочете `--target es5`, але при цьому використовувати найновіші функції, такі як `Promise`. Для підтримки цього ви можете отримати явний контроль над `lib` за допомогою опції компілятора `lib`.

> Примітка: використання `--lib` відокремлює будь-яку магію lib від `--target`, надаючи вам кращий контроль.

Ви можете надати цю опцію в командному рядку або в `tsconfig.json` (рекомендовано):

**Command line**:
```
tsc --target es5 --lib dom,es6
```
**tsconfig.json**:
```json
"compilerOptions": {
    "lib": ["dom", "es6"]
}
```

Бібліотеки можна класифікувати наступним чином:

* Масова функція JavaScript:
  * es5
  * es6
  * es2015
  * es7
  * es2016
  * es2017
  * esnext
* Середовище виконання
  * дом
  * дом.ітераційний
  * веб-працівник
  * scripthost
* ESNext By-Feature Options (навіть менше, ніж групова функція)
  * es2015.core
  * es2015.collection
  * es2015.generator
  * es2015.iterable
  * es2015.promise
  * es2015.proxy
  * es2015.reflect
  * es2015.symbol
  * es2015.symbol.wellknown
  * es2016.array.include
  * es2017.object
  * es2017.sharedmemory
  * esnext.asynciterable

> ПРИМІТКА: параметр `--lib` забезпечує надзвичайно тонке керування. Тож ви, швидше за все, захочете вибрати предмет із категорій масово + середовище.
> Якщо --lib не вказано, буде введено бібліотеку за замовчуванням:

* Для --target es5 => es5, dom, scripthost
* Для --target es6 => es6, dom, dom.iterable, scripthost

Моя особиста рекомендація:

```json
"compilerOptions": {
    "target": "es5",
    "lib": ["es6", "dom"]
}
```

Приклад із символом ES5:

Symbol API не включається, якщо ціль — es5. Насправді ми отримуємо помилку на зразок: [ts] Cannot find name 'Symbol'.
Ми можемо використовувати «target»: «es5» у поєднанні з «lib», щоб надати Symbol API у TypeScript:

```json
"compilerOptions": {
    "target": "es5",
    "lib": ["es5", "dom", "scripthost", "es2015.symbol"]
}
```

## Polyfill for old JavaScript engines

> [Egghead PRO Video on this subject](https://egghead.io/lessons/typescript-using-es6-and-esnext-with-typescript)

Існує чимало функцій середовища виконання, таких як `Map` / `Set` і навіть `Promise` (цей список, звичайно, буде змінюватися з часом), які ви можете використовувати з сучасними параметрами `lib`. Щоб використовувати їх, все, що вам потрібно зробити, це використовувати `core-js`. Просто встановіть:

```
npm install core-js --save-dev
```
And add an import to your application entry point: 

```js
import "core-js";
```

І він має заповнити ці функції середовища виконання для вас 🌹.
