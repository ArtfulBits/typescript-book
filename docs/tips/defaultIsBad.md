## `export default` concerns
Недолікі `export default`

Вважайте, що у вас є файл `foo.ts` з контентом:

```ts
class Foo {
}
export default Foo;
```

Ви імпортуєте його (in `bar.ts`) з допомогою синтаксису ES6:

```ts
import Foo from "./foo";
```

Маємо декілька проблем:
* Якщо ви змінюєте `Foo` в `foo.ts` це не буде зміненно в `bar.ts`.
* Якщо вам знадобиться експортувати більше речей із `foo.ts` (це те, що буде у багатьох ваших файлів), тоді вам доведеться жонглювати синтаксисом імпорту.

Тому я рекомендую легкий імпорт + деструктурований імпорт. Наприклад `foo.ts`:

```ts
export class Foo {
}
```
Далі:

```ts
import { Foo } from "./foo";
```

Нижче наведени інши причини.

### Poor Discoverability
Погана наочність.

Для експорту за замовченням притаманна дуже низька наочність. Вы не можете исследовать модуль с помощью IntelliSense, чтобы узнать, есть ли у него экспорт по умолчанию или нет.

З експортом за замовчуванням ви нічого тут не отримаєте (можливо, він експортує за замовчуванням / можливо, ні `¯\_(ツ)_/¯`):
```
import /* here */ from 'something';
```

Без експорту за замовчуванням ви отримуєте гарну підказку: 

```
import { /* here */ } from 'something';
```

### Autocomplete 
Автодоповнення.

Незалежно від того, чи знаєте ви про експорт, ви навіть виконуєте автозаповнення `import {/*here*/} from "./foo";` в місці курсору. Надає вашим розробникам трохи полегшення для рук.

### CommonJS interop
З `default` є жахливий досвід для користувачів звичайного JS, яким доводиться `const {default} = require('module/foo');` замість `const {Foo} = require('module/foo')`. Швидше за все, ви захочете перейменувати експорт `default` на щось інше під час його імпорту.

### Typo Protection
Захист типів.

Ви не отримаєте помилок, наприклад, коли один розробник виконує `import Foo from "./foo";`, а інший виконує `import foo from "./foo";`

### TypeScript auto-import
AШвидке виправлення автоматичного імпорту працює краще. Ви використовуєте `Foo`, і автоматичний імпорт запише `import { Foo } з "./foo";`, оскільки це чітко визначене ім’я, експортоване з модуля. Деякі інструменти намагаються магічно прочитати та *вивести* ім’я для експорту за замовчуванням, але магія нестійка.

### Re-exporting
Реекспортінг.

Повторний експорт є звичайним для кореневого файлу `index` в пакетах npm і змушує вас вручну назвати експорт за замовчуванням, наприклад, `export { default as Foo } from "./foo";` (із замовчуванням) проти `export * from "./foo"` (з названими експортами).

### Dynamic Imports
Динамічний імпорт.

Експорт за замовчуванням викриває себе погано названим `default` у динамічному `import`, наприклад.

```ts
const HighCharts = await import('https://code.highcharts.com/js/es-modules/masters/highcharts.src.js');
HighCharts.default.chart('container', { ... }); // Notice `.default`
```

Набагато краще так: 

```ts
const {HighCharts} = await import('https://code.highcharts.com/js/es-modules/masters/highcharts.src.js');
HighCharts.chart('container', { ... }); // Notice `.default`
```


### Needs two lines for non-class / non-function
Потрібні два рядки для не-класу/не-функції.

Може бути одним оператором для функції/класу, наприклад:

```ts
export default function foo() {
}
```

Може бути одним оператором для *non named / type annotated* об’єктів, наприклад.

```ts
export default {
  notAFunction: 'Yeah, I am not a function or a class',
  soWhat: 'The export is now *removed* from the declaration'
};
```

Але в іншому випадку потрібні два твердження:
```ts
// Якщо вам потрібно назвати його (тут `foo`) для локального використання АБО потрібно примітити тип (тут `Foo`)
const foo: Foo = {
  notAFunction: 'Yeah, I am not a function or a class',
  soWhat: 'The export is now *removed* from the declaration'
};
export default foo;
```
