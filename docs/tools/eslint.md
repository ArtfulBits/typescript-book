# ESLint

ESLint існував для лінтування JavaScript, але тепер він також стає дефакто лінтером для [TypeScript](https://github.com/Microsoft/TypeScript/issues/29288), завдяки 
[співпраці](https://eslint.org/blog/2019/01/future-typescript-eslint) між двома командами.

## Install
Встановлення

Щоб налаштувати ESLint для TypeScript, вам потрібні такі пакети:

```sh
npm i eslint eslint-plugin-react @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

> TIP: eslint називає свої пакети з правилами "plugin"

* eslint : Core eslint 
* eslint-plugin-react : Правіла для react від eslint. [Supported rules list](https://github.com/yannickcr/eslint-plugin-react#list-of-supported-rules)
* @typescript-eslint/parse : Дозволяє eslint розуміти ts / tsx files 
* @typescript-eslint/eslint-plugin : Для правил TypeScript. [Supported rules list](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin#supported-rules)

> Как ви бачите, 2 eslint пакети (для використання js та ts) та 2 @typescript-eslint пакета (для ts). Така звишена увага для TypeScript не *that much*.

## Configure 
Конфігурація

Створіть `.eslintrc.js`:

```js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: './tsconfig.json',
  },
  plugins: ['@typescript-eslint'],
  extends: [
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
  ],
  rules:  {
  // Правила перезапису, указані в розширених конфігураціях, наприклад
   // "@typescript-eslint/explicit-function-return-type": "off",
}
```

## Run
Використання

У вашому `package.json` додайте до `scripts`:

```json
{
  "scripts": {
    "lint": "eslint \"src/**\""
  }
}
```

Тепер ви можете `npm run lint` для перевірки.

## Configure VSCode 
Конфігурація VSCode 

* Встановіть розширення https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint

* Додайте до `settings.json`:
```js
"eslint.validate":  [
  "javascript",
  "javascriptreact",
  {"language":  "typescript",  "autoFix":  true  },
  {"language":  "typescriptreact",  "autoFix":  true  }
],
```
