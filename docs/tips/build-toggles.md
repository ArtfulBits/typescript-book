## Перемикачі збірки

У JavaScript-проектах зазвичай потрібно перемикатися в залежності від того, де вони запущені. Ви можете зробити це досить легко з webpack, оскільки він підтримує *видалення мертвого коду* на основі змінних середовища.

Додайте різні цілі в `scripts` вашого `package.json`:

```json
"build:test": "webpack -p --config ./src/webpack.config.js",
"build:prod": "webpack -p --define process.env.NODE_ENV='\"production\"' --config ./src/webpack.config.js",
```

Звичайно, я припускаю, що ви встановили `npm install webpack --save-dev`. Тепер ви можете запустити `npm run build:test` тощо.

Використання цієї змінної також дуже просте:

```ts
/**
 * Цей інтерфейс переконується, що ми не пропустимо додавання властивості до `prod` та `test`
 */
interface Config {
  someItem: string;
}

/**
 * Ми експортуємо лише одну річ. Конфігурацію.
 */
export let config: Config;

/**
 * Визначення `process.env.NODE_ENV` залежить від webpack
 *
 * Весь блок `else` буде видалено в емітованому JavaScript
 * для продакшен-збірки
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

> Ми використовуємо `process.env.NODE_ENV` лише тому, що це звичайно в багатьох бібліотеках JavaScript, наприклад, `React`.