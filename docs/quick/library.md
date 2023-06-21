# Creating TypeScript node modules

* [Урок зі створення вузлових модулів TypeScript](https://egghead.io/lessons/typescript-create-high-quality-npm-packages-using-typescript)

Користуватися модулями, написаними на TypeScript, надзвичайно весело, оскільки ви отримуєте чудову безпеку під час компіляції та автозаповнення (по суті, виконувану документацію).

Модулі TypeScript можна використовувати як у браузері nodejs (як є) (з чимось на зразок webpack).

Створити високоякісний модуль TypeScript просто. Припустімо таку бажану структуру папок для вашого пакета:

```text
package
├─ package.json
├─ tsconfig.json
├─ src
│  ├─ index.ts
│  ├─ foo.ts
│  └─ ...All your source files (Authored)
└─ lib
  ├─ index.d.ts.map
  ├─ index.d.ts
  ├─ index.js
  ├─ foo.d.ts.map
  ├─ foo.d.ts
  ├─ foo.js
  └─ ... All your compiled files (Generated)
```

* `src/index.ts`: тут ви експортуєте все, що очікуєте, що буде спожито з вашого проекту. Наприклад, `export { Foo } from './foo';`. Експорт із цього файлу робить його доступним для споживання, коли хтось виконує `імпорт { /* Тут */ } із 'example';`

* У вашому `tsconfig.json`
   * мають `compilerOptions`: `"outDir": "lib"` + `"declaration": true` + `"declarationMap" : true` < Це створює `.js` (JavaScript) `.d.ts` (декларації для TypeSafety) і `.d.ts.map` (вмикає `declaration .d.ts` => `source .ts` навігації IDE) у папці lib.
   * мають `include: ["src"]` < Це включає всі файли з каталогу `src`.

* У вашому `package.json` є
   * `"main": "lib/index"` < Це вказує завантажити `lib/index.js` для коду виконання.
   * `"types": "lib/index"` < Це повідомляє TypeScript завантажити `lib/index.d.ts` для перевірки типу.

Приклад пакета:
* `npm install typestyle` [для TypeStyle](https://www.npmjs.com/package/typestyle)
* Використання: `import { style } from 'typestyle';` буде повністю безпечним.

### Managing Dependencies
#### devDependencies
* Якщо ваш пакет залежить від іншого пакета під час його розробки (наприклад, `prettier`), ви повинні встановити їх як `devDependency`. Таким чином вони не забруднюватимуть `node_modules` споживачів вашого модуля (оскільки `npm i foo` не встановлює `devDependencies` `foo`).
* `typescript` зазвичай є `devDependency`, оскільки ви використовуєте його лише для створення свого пакета. Споживачі можуть використовувати ваш пакет із або без TypeScript.
* Якщо ваш пакет залежить від інших пакетів, створених JavaScript, і ви хочете використовувати його з безпекою типів у своєму проекті, розмістіть їхні типи (наприклад, `@types/foo`) у `devDependencies`. Типами JavaScript слід керувати *out of bound* з основних потоків NPM. Екосистема JavaScript надто часто порушує типи без семантичного керування версіями, тому, якщо вашим користувачам потрібні типи для них, вони повинні встановити версію @types/foo, яка їм підходить. Якщо ви хочете, щоб користувачі встановили ці типи, ви можете розмістити їх у `peerDependencies`, згаданих далі.

#### peerDependencies
Якщо ваш пакет залежить від пакета, з яким він *works with* (на відміну від *works using*), наприклад. `react`, помістіть їх у `peerDependencies` так само, як і з необробленими пакетами JS. Щоб перевірити їх локально, ви також повинні помістити їх у `devDependencies`.

Тепер:
* Коли ви розробляєте пакет, ви отримаєте версію залежності, яку ви вказали у своїх `devDependencies`.
* Коли хтось встановлює ваш пакет, він *not* отримає цю залежність (оскільки `npm i foo` не встановлює `devDependencies` з `foo`), але вони отримають попередження про те, що їм слід встановити відсутні `peerDependencies` вашого пакета .

#### dependencies
Якщо ваш пакунок *обертае* інший пакунок (призначений для внутрішнього використання навіть після компіляції), вам слід помістити їх у `залежності`. Тепер, коли хтось встановить ваш пакет, він отримає ваш пакет + будь-які його залежності.
