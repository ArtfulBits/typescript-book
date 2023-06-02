# TypeScript з Node.js
TypeScript має *першокласну* підтримку Node.js з моменту свого створення. Ось як налаштувати швидкий проект Node.js:

> Примітка: багато з цих кроків насправді є загальноприйнятими кроками налаштування Node.js

1. Налаштуйте проект Node.js `package.json`. Швидкий варіант: `npm init -y`
1. Додайте TypeScript (`npm install typescript --save-dev`)
1. Додайте `node.d.ts` (`npm install @types/node --save-dev`)
1. Ініціалізуйте `tsconfig.json` для налаштування TypeScript з декількома ключовими параметрами у вашому tsconfig.json (`npx tsc --init --rootDir src --outDir lib --esModuleInterop --resolveJsonModule --lib es6,dom  --module commonjs`)

Це все! Запустіть свій редактор коду (наприклад, `code .`) та поекспериментуйте. Тепер ви можете використовувати всі вбудовані модулі Node (наприклад, `import * as fs from 'fs';`) з усіма перевагами безпеки та ергономіки розробника TypeScript!

Весь ваш код TypeScript знаходиться в `src`, а згенерований JavaScript - в `lib`.

## Бонус: Жива компіляція + запуск
* Додайте `ts-node`, який ми використовуватимемо для живої компіляції та запуску в Node (`npm install ts-node --save-dev`)
* Додайте `nodemon`, який буде викликати `ts-node` кожного разу, коли змінюється файл (`npm install nodemon --save-dev`)

Тепер просто додайте ціль `script` до вашого `package.json` на основі вхідного файлу вашої програми, наприклад, припускаючи, що це `index.ts`:

```json
  "scripts": {
    "start": "npm run build:live",
    "build": "tsc -p .",
    "build:live": "nodemon --watch 'src/**/*.ts' --exec \"ts-node\" src/index.ts"
  },
```

Тепер ви можете запустити `npm start`, і коли ви редагуєте `index.ts`:

* nodemon перезапускає свою команду (ts-node)
* ts-node автоматично транспілює, використовуючи tsconfig.json та встановлену версію TypeScript,
* ts-node запускає вихідний JavaScript через Node.js.

А коли ви готові розгорнути свою JavaScript-програму, запустіть `npm run build`.

## Бонусні бали

Такі модулі NPM працюють прекрасно з browserify (з використанням tsify) або webpack (з використанням ts-loader).