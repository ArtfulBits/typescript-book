# JSX без React

[![DesignTSX](https://raw.githubusercontent.com/basarat/typescript-book/master/images/designtsx-banner.png)](https://designtsx.com)

TypeScript надає вам можливість використовувати щось інше, крім React, з JSX  у безпечний для типів спосіб. Нижче перераховані пункти, що стосуються налаштування, проте зверніть увагу, що це для досвідчених авторів фреймворків інтерфейсів:

* Ви можете вимкнути генерацію коду у стилі `react`, встановивши опцію `"jsx": "preserve"`. Це означає, що JSX буде виводитись *як є*, а потім ви можете використовувати власний транспілятор для трансляції частин JSX.
* Використання глобального модуля `JSX`:
    * Ви можете контролювати, які HTML-теги доступні і як вони перевіряються на тип налаштовуючи члени інтерфейсу `JSX.IntrinsicElements`.
    * При використанні компонентів:
        * Ви можете керувати тим, який `клас` має бути успадкований компонентами, налаштовуючи дефолтний інтерфейс `interface ElementClass extends React.Component<any, any> { }`.
        * Ви можете керувати тим, яка властивість використовується для перевірки типу атрибутів (за дефолтом це `props`) налаштувавши оголошення `declare module JSX { interface ElementAttributesProperty { props: {}; } }`.

## `jsxFactory`

Передача `--jsxFactory <Ім'я фабрики JSX>` разом з `--jsx react` дозволяє використовувати іншу фабрику JSX, відмінну від стандартної `React`.

Ім'я нової фабрики буде використовуватися для виклику функцій `createElement`.

### Приклад

```ts
import {jsxFactory} from "jsxFactory";

var div = <div>Hello JSX!</div>
```

Скомпільовано з:

```shell
tsc --jsx react --reactNamespace jsxFactory --m commonJS
```

Результатом є:

```js
"use strict";
var jsxFactory_1 = require("jsxFactory");
var div = jsxFactory_1.jsxFactory.createElement("div", null, "Hello JSX!");
```

## `jsx` pragma

Ви навіть можете вказати різні `jsxFactory` для кожного файлу за допомогою `jsxPragma`, наприклад. 

```js
/** @jsx jsxFactory */
import {jsxFactory} from "jsxFactory";

var div = <div>Hello JSX!</div>
```

За допомогою опції `--jsx react`, цей файл буде генерувати використання фабрики, вказаної в pragma JSX:

```js
"use strict";
var jsxFactory_1 = require("jsxFactory");
var div = jsxFactory_1.jsxFactory.createElement("div", null, "Hello JSX!");
```
