# global.d.ts

Ми обговорювали *global* та *file* модулі під час розгляду [проектів](./modules.md) і рекомендували використовувати файлові модулі та не забруднювати глобальний простір імен.

Тим не менше, якщо у вас є розробники-початківці TypeScript, ви можете надати їм файл `global.d.ts` для розміщення інтерфейсів/типів у глобальному просторі імен, щоб зробити деякі *types* просто *magically* доступним для використання в *цілоту* вашему коді TypeScript.

Іншим варіантом використання файлу `global.d.ts` є оголошення констант часу компіляції, які Webpack додає у вихідний код за допомогою стандартного [DefinePlugin](https://webpack.js.org/plugins/define -plugin/) плагін.

```ts
declare const BUILD_MODE_PRODUCTION: boolean; // can be used for conditional compiling
declare const BUILD_VERSION: string;
```

> Для будь-якого коду, який збирається генерувати *JavaScript*, ми наполегливо рекомендуємо використовувати *file modules* та використовувати лише `global.d.ts` для оголошення констант часу компіляції та/або для розширення стандартних декларацій типів, оголошених у `lib.d.ts`.

* Бонус: файл `global.d.ts` також добре підходить для швидкого `declare module  "some-library-you-dont-care-to-get-defs-for";` під час міграції JS до TS.
