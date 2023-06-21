# TypeScript with Node.js
TypeScript має *first class* підтримку Node.js з самого початку. Ось як налаштувати швидкий проект Node.js:

> Примітка: багато з цих кроків насправді є звичайними кроками налаштування Node.js

1. Встановіть `package.json` в проект Node.js. Швидкий шлях: `npm init -y`
1. Додайте TypeScript (`npm install typescript --save-dev`)
1. Додайте `node.d.ts` (`npm install @types/node --save-dev`)
1. Ініціалізуйте `tsconfig.json` для параметрів TypeScript з кількома ключовими параметрами у вашому tsconfig.json (`npx tsc --init --rootDir src --outDir lib --esModuleInterop --resolveJsonModule --lib es6,dom  --module commonjs`)

Це воно! Запустіть свою IDE (наприклад, `code .`) і спробуйте. Тепер ви можете використовувати всі вбудовані модулі вузлів (наприклад, `import * as fs from 'fs';`) з усією безпекою та ергономікою розробника TypeScript!

Весь ваш код TypeScript міститься в `src`, а згенерований JavaScript — у `lib`.

## Bonus: Live compile + run
* Додайте `ts-node`, який ми будемо використовувати для живої компіляції + запуску у вузлі (`npm install ts-node --save-dev`)
* Додайте `nodemon`, який буде викликати `ts-node` щоразу, коли файл буде змінено (`npm install nodemon --save-dev`)

Тепер просто додайте `script` до свого `package.json` на основі запису вашої програми, наприклад, припускаючи його `index.ts`:

```json
  "scripts": {
    "start": "npm run build:live",
    "build": "tsc -p .",
    "build:live": "nodemon --watch 'src/**/*.ts' --exec \"ts-node\" src/index.ts"
  },
```

Тепер ви можете запустити `npm start` і під час редагування `index.ts`:

* nodemon повторно виконує свою команду (ts-node)
* ts-node transpiles автоматично збирає tsconfig.json і встановлену версію TypeScript,
* ts-node запускає вихідний JavaScript через Node.js.

І коли ви будете готові розгорнути свою програму JavaScript, запустіть `npm run build`.


## Bonus points

Такі модулі NPM чудово працюють із browserify (за допомогою tsify) або webpack (за допомогою ts-loader).
