## Modules

### Global Module

За замовчуванням, коли ви починаєте вводити код у новому файлі TypeScript, ваш код знаходиться в *глобальному* просторі імен. Як демонстрацію розглянемо файл `foo.ts`:

```ts
var foo = 123;
```

Якщо ви тепер створите *новий* файл `bar.ts` у тому самому проекті, вам буде *дозволено* системою типів TypeScript використовувати змінну `foo` так, якби вона була доступна глобально:

```ts
var bar = foo; // allowed
```
Зайве говорити, що мати глобальний простір імен небезпечно, оскільки це відкриває ваш код для конфліктів імен. Ми рекомендуємо використовувати файлові модулі, які представлені далі.

### File Module
Також називаються *зовнішніми модулями*. Якщо у вас є «імпорт» або «експорт» на кореневому рівні файлу TypeScript, тоді в цьому файлі створюється *локальна* область. Отже, якщо ми змінимо попередній `foo.ts` на такий (зверніть увагу на використання `export`):

```ts
export var foo = 123;
```

Ми більше не матимемо `foo` у глобальному просторі імен. Це можна продемонструвати, створивши новий файл `bar.ts` наступним чином:
```ts
var bar = foo; // ERROR: "cannot find name 'foo'"
```

Якщо ви хочете використовувати матеріали з `foo.ts` у `bar.ts` *вам потрібно явно імпортувати це*. Це показано в оновленому `bar.ts` нижче:

```ts
import { foo } from "./foo";
var bar = foo; // allowed
```
Використання `import` у `bar.ts` не тільки дає змогу вводити матеріали з інших файлів, але також позначає файл `bar.ts` як *модуль*, а отже, оголошення в `bar.ts` не також не забруднювати глобальний простір імен.

Те, що JavaScript генерується з даного файлу TypeScript, який використовує зовнішні модулі, керується прапором компілятора під назвою `module`.
