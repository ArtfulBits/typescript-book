## External modules
Шаблон зовнішнього модуля TypeScript містить велику потужність і зручність використання. Тут ми обговорюємо його потужність і деякі моделі, необхідні для відображення використання в реальному світі.

### Clarification: commonjs, amd, es modules, others

Спочатку нам потрібно прояснити (жахливу) невідповідність систем модулів. Я просто дам вам свою *поточну* рекомендацію та видалю шум, тобто не покажу всі *інші* способи роботи.

З *того самого TypeScript* ви можете генерувати різний *JavaScript* залежно від параметра `module`. Ось речі, які ви можете ігнорувати (мені не цікаво пояснювати мертві технології):

* AMD: не використовувати. Був лише браузер.
* SystemJS: це був хороший експеримент. Замінено модулями ES.
* Модулі ES: ще не готові.

Тепер це лише варіанти для *генерування JavaScript*. Замість цих параметрів використовуйте `module:commonjs`

Те, як ви *пишете* модулі TypeScript, також є дещо безладним. Ось як цього не робити *сьогодні*:

* `import foo = require('foo')`. тобто `import/require`. Натомість використовуйте синтаксис модуля ES.

Круто, покінчивши з цим, давайте розглянемо синтаксис модуля ES.

> Резюме: Використовуйте `module:commonjs` і використовуйте синтаксис модуля ES для імпорту/експорту/авторських модулів.

### ES Module syntax

* Експортувати змінну (або тип) так само просто, як додати префікс ключового слова `export`.

```js
// file `foo.ts`
export let someVar = 123;
export type SomeType = {
  foo: string;
};
```

* Експортування змінної або типу в спеціальному операторі `export`.

```js
// file `foo.ts`
let someVar = 123;
type SomeType = {
  foo: string;
};
export {
  someVar,
  SomeType
};
```
* Експортування змінної або типу в спеціальному операторі `export` *з перейменуванням*.

```js
// file `foo.ts`
let someVar = 123;
export { someVar as aDifferentName };
```

* Імпорт змінну або тип за допомогою `import`.

```js
// file `bar.ts`
import { someVar, SomeType } from './foo';
```

* Імпортуйте змінну або тип за допомогою `import` *з перейменуванням*.

```js
// file `bar.ts`
import { someVar as aDifferentName } from './foo';
```

* Імпортуйте все з модуля в назву за допомогою `import * as`, наприклад.

```js
// file `bar.ts`
import * as foo from './foo';
// you can use `foo.someVar` and `foo.SomeType` and anything else that foo might export.
```

* Імпортуйте файл *тільки* для його побічного ефекту за допомогою одного оператора імпорту:

```js
import 'core-js'; // a common polyfill library
```

* Повторний експорт усіх елементів з іншого модуля

```js
export * from './foo';
```

* Повторний експорт лише деяких елементів з іншого модуля

```js
export { someVar } from './foo';
```

* Повторний експорт лише деяких елементів з іншого модуля * з перейменуванням *

```js
export { someVar as aDifferentName } from './foo';
```

### Default exports/imports

Як ви дізнаєтесь пізніше, я не прихильник експорту за замовчуванням. Тим не менш, тут є синтаксис для експорту та використання стандартних експортів

* Експортуйте за допомогою `export default`
   * перед змінною (не потрібно `let / const / var`)
   * перед функцією
   * перед уроком

```js
// some var
export default someVar = 123;
// OR Some function
export default function someFunction() { }
// OR Some class
export default class SomeClass { }
```

* Імпортуйте за допомогою синтаксису `import someName from "someModule"` (ви можете назвати імпорт як завгодно).

```js
import someLocalNameForThisFile from "../foo";
```

### Module paths

>> Я просто припускаю `moduleResolution: "Node"`. Це параметр, який ви повинні мати у своїй конфігурації TypeScript. Цей параметр автоматично передбачається `module:commonjs`.

Є два різних типи модулів. Розрізнення визначається розділом шляху оператора імпорту (наприклад, `import foo from 'THIS IS THE PATH SECTION'`).

* Модулі відносного шляху (де шлях починається з `.`, наприклад `./someFile` або `../../someFolder/someFile` тощо)
* Інші модулі динамічного пошуку (наприклад, `'core-js'` або `'typestyle'` або `'react'` або навіть `'react/core'` тощо)

Основна відмінність полягає в тому, *як модуль вирішується у файловій системі*.

> Я буду використовувати концептуальний термін *місце*, який я поясню після згадки шаблону пошуку.

#### Relative path modules
Легко, просто дотримуйтеся відносного шляху :)

якщо файл `bar.ts` виконує `імпорт * як foo з './foo';`, тоді місце `foo` має існувати в тій же папці.
* якщо файл `bar.ts` виконує `імпорт * як foo з '../foo';`, тоді місце `foo` має існувати в папці вище.
* якщо файл `bar.ts` виконує `імпорт * як foo з '../someFolder/foo';`, тоді на одну папку вище, має бути папка `someFolder` з місцем `foo`

Або будь-який інший відносний шлях, який ви можете придумати :)

#### Dynamic lookup

Якщо шлях імпорту *не* відносний, пошук керується [*роздільністю стилю вузла*](https://nodejs.org/api/modules.html#modules_all_together). Тут я наведу лише простий приклад:

* У вас є `імпорт * як foo з 'foo'`, нижче наведено місця, які перевіряються *по порядку*
   * `./node_modules/foo`
   * `../node_modules/foo`
   * `../../node_modules/foo`
   * До кореня файлової системи

* У вас є `імпорт * як foo з 'something/foo'`, нижче наведено місця, які перевіряються *по порядку*
   * `./node_modules/something/foo`
   * `../node_modules/something/foo`
   * `../../node_modules/something/foo`
   * До кореня файлової системи


### What is *place*
Коли я кажу *місця, які перевіряються*, я маю на увазі, що в цьому місці перевіряються такі речі. Наприклад. для місця `foo`:

* Якщо місцем є файл, напр. `foo.ts`, ура!
* інакше, якщо місцем є папка і є файл `foo/index.ts`, ура!
* інакше, якщо місцем є папка і існує `foo/package.json` і файл, указаний у ключі `types` у package.json, тоді ура!
* інакше, якщо місцем є папка, і існує `package.json` і файл, указаний у ключі `main` у package.json, який існує, тоді ура!

Під файлом я насправді маю на увазі `.ts` / `.d.ts` і `.js`.

І це все. Тепер ви експерт із пошуку модулів (це не маленький подвиг!).

### Overturning dynamic lookup *just for types*
Ви можете оголосити модуль *глобально* для свого проекту за допомогою `declare module 'somePath'`, а потім імпорт буде *чарівним* чином вирішувати цей шлях

e.g.
```ts
// global.d.ts
declare module 'foo' {
  // Some variable declarations
  export var bar: number; /*sample*/
}
```

and then:
```ts
// anyOtherTsFileInYourProject.ts
import * as foo from 'foo';
// TypeScript assumes (without doing any lookup) that
// foo is {bar:number}

```

### `import/require` for importing type only
Наступна заява:

```ts
import foo = require('foo');
```

насправді робить *дві* речі:

* Імпортує інформацію про тип модуля foo.
* Визначає залежність часу виконання від модуля foo.

Ви можете вибирати так, щоб завантажувалася лише *інформація про тип* і не виникала залежність під час виконання. Перш ніж продовжити, можливо, ви захочете підсумувати розділ [*просторів для оголошень*](../project/declarationspaces.md) у книзі.

Якщо ви не використовуєте імпортоване ім’я в просторі оголошення змінних, імпорт буде повністю видалено зі згенерованого JavaScript. Це найкраще пояснити на прикладах. Як тільки ви це зрозумієте, ми представимо вам випадки використання.

#### Example 1
```ts
import foo = require('foo');
```
створить JavaScript:

```js

```
Це вірно. *Порожній* файл як foo не використовується.

#### Example 2
```ts
import foo = require('foo');
var bar: foo;
```
створить JavaScript:
```js
var bar;
```
Це тому, що `foo` (або будь-яка з його властивостей, наприклад `foo.bas`) ніколи не використовується як змінна.

#### Example 3
```ts
import foo = require('foo');
var bar = foo;
```
створить JavaScript (припускаючи, що commonjs):

```js
var foo = require('foo');
var bar = foo;
```
Це тому, що `foo` використовується як змінна.


### Use case: Lazy loading
Висновок типу потрібно зробити *заздалегідь*. Це означає, що якщо ви хочете використовувати певний тип із файлу `foo` у файлі `bar`, вам доведеться зробити:

```ts
import foo = require('foo');
var bar: foo.SomeType;
```
Однак ви можете завантажувати файл `foo` лише під час виконання за певних умов. У таких випадках вам слід використовувати імпортовану назву лише в *анотаціях типу*, а **не** як *змінну*. Це видаляє будь-який *попередній* код залежності часу виконання, який впроваджується TypeScript. Потім *вручну імпортуйте* фактичний модуль, використовуючи код, який відповідає вашому завантажувачу модулів.

Як приклад, розглянемо наступний код на основі `commonjs`, де ми завантажуємо модуль `'foo`` лише під час певного виклику функції:

```ts
import foo = require('foo');

export function loadFoo() {
    // This is lazy loading `foo` and using the original module *only* as a type annotation
    var _foo: typeof foo = require('foo');
    // Now use `_foo` as a variable instead of `foo`.
}
```

Схожий приклад на `amd` (using requirejs) буде:
```ts
import foo = require('foo');

export function loadFoo() {
    // This is lazy loading `foo` and using the original module *only* as a type annotation
    require(['foo'], (_foo: typeof foo) => {
        // Now use `_foo` as a variable instead of `foo`.
    });
}
```

Цей шаблон зазвичай використовується:
* у веб-додатках, де ви завантажуєте певний JavaScript за певними маршрутами,
* у вузлових програмах, де ви завантажуєте лише певні модулі, якщо це необхідно для прискорення завантаження програми.

### Use case: Breaking Circular dependencies

Подібно до випадку використання відкладеного завантаження, деякі завантажувачі модулів (commonjs/node і amd/requirejs) погано працюють із циклічними залежностями. У таких випадках корисно мати *ліниве завантаження* коду в одному напрямку та завантаження модулів наперед в іншому напрямку.

### Use case: Ensure Import

Іноді ви хочете завантажити файл лише для побічного ефекту (наприклад, модуль може зареєструватися в якійсь бібліотеці, наприклад [CodeMirror addons](https://codemirror.net/doc/manual.html#addons) тощо). Однак, якщо ви просто виконуєте команду `import/require`, транспільований JavaScript не міститиме залежності від модуля, і ваш завантажувач модуля (наприклад, веб-пакет) може повністю ігнорувати імпорт. У таких випадках ви можете використовувати змінну `ensureImport`, щоб переконатися, що скомпільований JavaScript приймає залежність від модуля, наприклад:

```ts
import foo = require('./foo');
import bar = require('./bar');
import bas = require('./bas');
const ensureImport: any =
    foo
    && bar
    && bas;
```
