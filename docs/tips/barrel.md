## Barrel

Barrel це спосіб згортання експорту з кількох модулів в один зручний модуль. Сам барель є файлом модуля, який повторно експортує вибрані експорти інших модулів.

Уявіть наступну структуру класів у бібліотеці: 

```ts
// demo/foo.ts
export class Foo {}

// demo/bar.ts
export class Bar {}

// demo/baz.ts
export class Baz {}
```
Без barrel користувач потребує зробити 3 import:

```ts
import { Foo } from '../demo/foo';
import { Bar } from '../demo/bar';
import { Baz } from '../demo/baz';
```

Замість цього можно використовувати barrel `demo/index.ts` який реекспортує: 

```ts
// demo/index.ts
export * from './foo'; // re-export all of its exports
export * from './bar'; // re-export all of its exports
export * from './baz'; // re-export all of its exports
```

Зараз можно використати простий синтансис:

```ts
import { Foo, Bar, Baz } from '../demo'; // demo/index.ts is implied
```

### Named exports
Іменований експорт.

Замість експорту всього через  `*`, ви можете вибрати модуль для експорту по назві. Наприклад, `baz.ts` має функції:

```ts
// demo/foo.ts
export class Foo {}

// demo/bar.ts
export class Bar {}

// demo/baz.ts
export function getBaz() {}
export function setBaz() {}
```

Якщо ви не хочете експортовать `getBaz` / `setBaz` з demoнатомість ви можете помістити їх у змінну, імпортувавши їх у назву та експортувавши цю назву, як показано нижче: 

```ts
// demo/index.ts
export * from './foo'; // re-export all of its exports
export * from './bar'; // re-export all of its exports

import * as baz from './baz'; // import as a name
export { baz }; // export the name
```

Зараз ви маєте такий імпорт: 

```ts
import { Foo, Bar, baz } from '../demo'; // demo/index.ts is implied

// usage
baz.getBaz();
baz.setBaz();
// etc. ...
```
