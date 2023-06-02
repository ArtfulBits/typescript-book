# Prettier

Prettier - це чудовий інструмент від Facebook, який робить форматування коду настільки простим, що варто згадати. Налаштування з TypeScript за допомогою нашої рекомендованої настройки проекту (тобто все в папці `src`) дуже просте:

## Налаштування

* `npm install prettier -D`
* Додайте `scripts` до `package.json`:

```
    "prettier:base": "prettier --parser typescript --single-quote",
    "prettier:check": "npm run prettier:base -- --list-different \"src/**/*.{ts,tsx}\"",
    "prettier:write": "npm run prettier:base -- --write \"src/**/*.{ts,tsx}\""
```

## Використання
На вашому сервері збірки:
* `npm run prettier:check`

Під час розробки (або перед комітом):
* `npm run prettier:write`