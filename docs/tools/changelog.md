## Зміни
> Читання файлу у форматі markdown з прогресом у проекті легше, ніж читання журналу комітів.

Автоматичне створення журналу змін з повідомлень про коміти є досить поширеним шаблоном в наші дні. Існує проект під назвою [conventional-changelog](https://github.com/conventional-changelog/conventional-changelog), який генерує журнал змін з повідомлень про коміти, що відповідають *конвенції*.

### Конвенція повідомлень про коміти
Найпоширенішою конвенцією є конвенція повідомлень про коміти *angular*, яка [детально описана тут](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines).

### Налаштування
* Встановити:

```bash
npm install standard-version -D
```

* Додати ціль `script` до вашого `package.json`:

```js
{
  "scripts": {
    "release": "standard-version"
  }
}
```

* Опціонально: Щоб автоматично надіслати новий *git commit and tag* та опублікувати в npm, додайте скрипт `postrelease`:

```js
{
  "scripts": {
    "release": "standard-version",
    "postrelease": "git push --follow-tags origin master && npm publish"
  }
}
```

### Реліз

Просто запустіть:

```bash
npm run release
```

На основі повідомлень про коміти автоматично визначається `major` | `minor` | `patch`. Щоб *явно* вказати версію, ви можете вказати `--release-as`, наприклад:

```bash
npm run release -- --release-as minor
```