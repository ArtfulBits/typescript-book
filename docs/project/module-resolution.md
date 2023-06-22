# TypeScript Module Resolution

Роздільна здатність TypeScript намагається змоделювати та підтримувати реальні системи/завантажувачі модулів (commonjs/nodejs, amd/requirejs, ES6/systemjs тощо). Найпростішим пошуком є відносний пошук шляху до файлу. Після цього все стає дещо складнішим *because of the nature of magical module loading done by various module loaders*.

## File Extensions

Ви імпортуєте такі модулі, як `foo` або `./foo`. Для будь-якого пошуку шляху до файлу TypeScript автоматично перевіряє файл `.ts`, `.d.ts`, `.tsx`, `.js` (необов’язково) або `.jsx` (необов’язково) у правильному порядку залежно від контексту . Ви **не** повинні вказувати розширення файлу з назвою модуля (без `foo.ts`, лише `foo`).

## Relative File Module

Імпорт із відносним шляхом, наприклад:

```ts
import foo = require('./foo');
```

Вказує компілятору TypeScript шукати файл TypeScript у відносному місці, наприклад. `./foo.ts` щодо поточного файлу. У цьому виді імпорту більше немає магії. Звичайно, це може бути довший шлях, напр. `./foo/bar/bas` або `../../../foo/bar/bas`, як і будь-які інші *relative paths*, до яких ви звикли на диску.

## Named Module

Наступна заява:

```ts
import foo = require('foo');
```

Вказує компілятору TypeScript шукати зовнішній модуль у такому порядку:

* Іменована [декларація модуля](#module-declaration) з файлу, який уже знаходиться в контексті компіляції.
* Якщо все ще не вирішено, а ви компілюєте за допомогою `--module commonjs` або встановили `--moduleResolution node`, тоді його шукають за допомогою алгоритму розв’язання [*node modules*](#node-modules).
* Якщо все ще не вирішено, і ви вказали `baseUrl` (і додатково `paths`), тоді спрацьовує алгоритм вирішення [*заміни шляху*](#path-substitutions).

Зауважте, що "foo" може бути довшим рядком шляху, наприклад. `"foo/bar/bas"`. Ключовим тут є те, що *it does not start with `./` or `../`*.

## Module Declaration

Оголошення модуля виглядає так:

```ts
declare module "foo" {

    /// Some variable declarations

    export var bar:number; /*sample*/
}
```

Це робить модуль `"foo"` *імпортованим*.

## Node Modules
Роздільна здатність node модуля фактично майже така ж, яку використовує Node.js / NPM ([офіційні документи nodejs](https://nodejs.org/api/modules.html#modules_all_together)). Ось проста ментальна модель, яку я маю:

* модуль `foo/bar` виведе на певний файл: `node_modules/foo` (модуль) + `foo/bar`

## Path Substitutions

TODO.

[//Comment1]:https://github.com/Microsoft/TypeScript/issues/2338
[//Comment2]:https://github.com/Microsoft/TypeScript/issues/5039
[//Comment3ExampleRedirectOfPackageJson]:https://github.com/Microsoft/TypeScript/issues/8528#issuecomment-219172026
[//Coment4ModuleResolutionInHandbook]:https://github.com/Microsoft/TypeScript-Handbook/blob/release-2.0/pages/Module%20Resolution.md#base-url
