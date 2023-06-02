# Non React JSX

[![DesignTSX](https://raw.githubusercontent.com/basarat/typescript-book/master/images/designtsx-banner.png)](https://designtsx.com)

TypeScript дозволяє використовувати щось інше, крім React з JSX в безпечному типовому режимі. Наступне перераховує точки налаштування, але зверніть увагу, що це для авторів розширених фреймворків інтерфейсу користувача:

* Ви можете вимкнути емісію стилю `react`, використовуючи опцію `"jsx": "preserve"`. Це означає, що JSX емітується *як є*, а потім ви можете використовувати свій власний транспілер для транспіляції частин JSX.
* Використовуючи глобальний модуль `JSX`:
    * Ви можете контролювати, які HTML-теги доступні та як вони перевіряються за типом, налаштовуючи члени інтерфейсу `JSX.IntrinsicElements`.
    * При використанні компонентів:
        * Ви можете контролювати, який `class` має успадковуватися компонентами, налаштовуючи типовий запис за замовчуванням `interface ElementClass extends React.Component<any, any> { }`.
        * Ви можете контролювати, яка властивість використовується для перевірки типу атрибутів (за замовчуванням `props`), налаштовуючи типовий запис `declare module JSX { interface ElementAttributesProperty { props: {}; } }`.

## `jsxFactory`

Передача `--jsxFactory <JSX factory Name>` разом з `--jsx react` дозволяє використовувати іншу фабрику JSX, ніж за замовчуванням `React`.

Нове ім'я фабрики буде використовуватися для виклику функцій `createElement`.

### Приклад

```ts
import {jsxFactory} from "jsxFactory";

var div = <div>Hello JSX!</div>
```

Скомпільовано з:

```shell
tsc --jsx react --reactNamespace jsxFactory --m commonJS
```

Результат:

```js
"use strict";
var jsxFactory_1 = require("jsxFactory");
var div = jsxFactory_1.jsxFactory.createElement("div", null, "Hello JSX!");
```

## `jsx` pragma

Ви можете навіть вказати різні `jsxFactory` для кожного файлу, використовуючи `jsxPragma`, наприклад:


```js
/** @jsx jsxFactory */
import {jsxFactory} from "jsxFactory";

var div = <div>Hello JSX!</div>
```

З `--jsx react` цей файл буде емітуватися з використанням фабрики, вказаної в jsx pragma:
```js
"use strict";
var jsxFactory_1 = require("jsxFactory");
var div = jsxFactory_1.jsxFactory.createElement("div", null, "Hello JSX!");
```