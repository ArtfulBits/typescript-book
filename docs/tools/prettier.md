# Prettier 

Prettier — це чудовий інструмент від Facebook, який робить форматування коду настільки простим, що варто згадати про нього. Налаштувати TypeScript за допомогою наших рекомендованих налаштувань проекту (також відомого як усе в папці `src`) надзвичайно просто:

## Setup 
Наставлення

* `npm install prettier -D` 
* Add `scripts` to `package.json`: 

```
    "prettier:base": "prettier --parser typescript --single-quote",
    "prettier:check": "npm run prettier:base -- --list-different \"src/**/*.{ts,tsx}\"",
    "prettier:write": "npm run prettier:base -- --write \"src/**/*.{ts,tsx}\""
```

## Usage 
Використання

На вашому сервері збірки:
* `npm run prettier:check` 

Під час розробки (або перед комітом):
* `npm run prettier:write`
