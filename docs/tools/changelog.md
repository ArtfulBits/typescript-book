## Changelog 
> Читати файл розмітки з прогресом у проекті легше, ніж читати журнал фіксації.

Автоматичне створення журналу змін із повідомлень комітів є досить поширеним шаблоном у наш час. Є проект під назвою [conventional-changelog](https://github.com/conventional-changelog/conventional-changelog) який генерує журнал змін із повідомлень про фіксацію, які відповідають *convention*.

### Commit message convention
Найпоширенішою угодою є угода про повідомлення фіксації  *angular*, яка  [детально описана тут . Налаштування](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines).

### Setup

* встановити:

```bash
npm install standard-version -D
```

* Додайте ціль  `script` до свого `package.json`:

```js
{
  "scripts": {
    "release": "standard-version"
  }
}
```

* Додатково: щоб автоматично надсилати новий  *git commit and tag* плюс публікацію до npm, додайте `postrelease` сценарій:

```js
{
  "scripts": {
    "release": "standard-version",
    "postrelease": "git push --follow-tags origin master && npm publish"
  }
}
```

### Releasing 

Простий запуск: 

```bash
npm run release
```

На основі повідомлень тип версії:  `major` | `minor` | `patch` визначається автоматично. Щоб *explicitly* явно вказати версію, ви можете вказати `--release-as` наприклад::

```bash
npm run release -- --release-as minor
```
