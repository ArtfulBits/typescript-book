## Build Toggles
Перемикачі побудови

Зазвичай проекти JavaScript перемикаються залежно від того, де вони виконуються. Ви можете зробити це досить легко за допомогою webpack, оскільки він підтримує *усунення мертвого коду* на основі змінних середовища.

Ви можете додати це у ваш `package.json` `scripts`:

```json
"build:test": "webpack -p --config ./src/webpack.config.js",
"build:prod": "webpack -p --define process.env.NODE_ENV='\"production\"' --config ./src/webpack.config.js",
```

Звичайно, я припускаю, що у вас є `npm install webpack --save-dev`. Зараз ви можете використовувати `npm run build:test`.

Використовувати цю змінну також дуже просто:

```ts
/**
 * This interface makes sure we don't miss adding a property to both `prod` and `test`
 */
interface Config {
  someItem: string;
}

/**
 * We only export a single thing. The config.
 */
export let config: Config;

/**
 * `process.env.NODE_ENV` definition is driven from webpack
 *
 * The whole `else` block will be removed in the emitted JavaScript
 *  for a production build
 */
if (process.env.NODE_ENV === 'production') {
  config = {
    someItem: 'prod'
  }
  console.log('Running in prod');
} else {
  config = {
    someItem: 'test'
  }
  console.log('Running in test');
}
```

> Ви використовуєте `process.env.NODE_ENV` просто тому, що це традиційно у багатьох бібліотеках JavaScript, наприклад, `React`.
