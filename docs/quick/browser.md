# TypeScript у браузері

[![DesignTSX](https://raw.githubusercontent.com/basarat/typescript-book/master/images/designtsx-banner.png)](https://designtsx.com)

Якщо ви використовуєте TypeScript для створення веб-додатка, ось мої рекомендації щодо швидкого налаштування проекту TypeScript + React (моєї улюбленої фреймворку для користувацького інтерфейсу).

## Загальне налаштування машини

* Встановіть [Node.js](https://nodejs.org/en/download/)
* Встановіть [Git](https://git-scm.com/downloads)

## Швидке налаштування проекту
Використовуйте [https://github.com/basarat/react-typescript](https://github.com/basarat/react-typescript) як базу.

```
git clone https://github.com/basarat/react-typescript.git
cd react-typescript
npm install
```

Тепер використовуйте його як базу та перейдіть до [розробки вашого неймовірного додатка](#develop-your-amazing-application)

## Детальне налаштування проекту
Якщо ви хочете дізнатися більше про деталі створення цього проекту (замість того, щоб використовувати його як базу), ось кроки, які потрібно виконати для його налаштування з нуля:

* Створіть директорію проекту:

```
mkdir your-project
cd your-project
```

* Створіть `tsconfig.json`:

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "module": "commonjs",
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "experimentalDecorators": true,
    "target": "es5",
    "jsx": "react",
    "lib": [
      "dom",
      "es6"
    ]
  },
  "include": [
    "src"
  ],
  "compileOnSave": false
}
```

* Створіть `package.json`.

```json
{
  "name": "react-typescript",
  "version": "0.0.0",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/basarat/react-typescript.git"
  },
  "scripts": {
    "build": "webpack -p",
    "start": "webpack-dev-server -d --content-base ./public"
  },
  "dependencies": {
    "@types/react": "16.4.10",
    "@types/react-dom": "16.0.7",
    "clean-webpack-plugin": "0.1.19",
    "html-webpack-plugin": "3.2.0",
    "react": "16.4.2",
    "react-dom": "16.4.2",
    "ts-loader": "4.4.2",
    "typescript": "3.0.1",
    "webpack": "4.16.5",
    "webpack-cli": "3.1.0",
    "webpack-dev-server": "3.1.5"
  }
}
```

* Створіть `webpack.config.js`, щоб зібрати ваші модулі в один файл `app.js`, який містить всі ваші ресурси:

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/app/app.tsx',
  plugins: [
    new CleanWebpackPlugin({
      cleanAfterEveryBuildPatterns: ['public/build']
    }),
    new HtmlWebpackPlugin({
      template: 'src/templates/index.html'
    }),
  ],
  output: {
    path: __dirname + '/public',
    filename: 'build/[name].[contenthash].js'
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js']
  },
  module: {
    rules: [
      { test: /\.tsx?$/, loader: 'ts-loader' }
    ]
  }
}
```

* Файл `src/templates/index.html`. Він буде використовуватися як шаблон для `index.html`, який генерується webpack. Згенерований файл буде в папці `public` та потім буде обслуговуватися вашим веб-сервером:

```html
<html>
  <body>
      <div id="root"></div>
  </body>
</html>

```

* `src/app/app.tsx`, який є точкою входу вашої фронтенд-додатка:

```js
import * as React from 'react';
import * as ReactDOM from 'react-dom';

const Hello: React.FunctionComponent<{ compiler: string, framework: string }> = (props) => {
  return (
    <div>
      <div>{props.compiler}</div>
      <div>{props.framework}</div>
    </div>
  );
}

ReactDOM.render(
  <Hello compiler="TypeScript" framework="React" />,
  document.getElementById("root")
);
```

# Розробляйте свій неймовірний додаток

> Ви можете отримати останні пакети, використовуючи `npm install typescript@latest react@latest react-dom@latest @types/react@latest @types/react-dom@latest webpack@latest webpack-dev-server@latest webpack-cli@latest ts-loader@latest clean-webpack-plugin@latest html-webpack-plugin@latest --save-exact`

* Виконуйте живу розробку, запустивши `npm start`.
    * Відвідайте [http://localhost:8080](http://localhost:8080)
    * Редагуйте `src/app/app.tsx` (або будь-який ts/tsx файл, який використовується якимось чином у `src/app/app.tsx`) та додаток живо перезавантажується.
    * Редагуйте `src/templates/index.html`, і сервер живо перезавантажується.
* Створіть продуктові ресурси, запустивши `npm run build`.
    * Обслуговуйте папку `public` (яка містить зібрані ресурси) з вашого сервера.