## Простори імен
Простори імен надають зручний синтаксис для спільного шаблону, що використовується в JavaScript:

```ts
(function(something) {

    something.foo = 123;

})(something || (something = {}))
```

Основна ідея полягає в тому, що `something || (something = {})` дозволяє анонімній функції `function(something) {}` *додавати елементи до існуючого об'єкту* (частина `something ||`) або *створювати новий об'єкт, а потім додавати елементи до цього об'єкту* (частина `|| (something = {})`). Це означає, що можна мати два таких блоки, розділені якоюсь межею виконання:

```ts
(function(something) {

    something.foo = 123;

})(something || (something = {}))

console.log(something); // {foo:123}

(function(something) {

    something.bar = 456;

})(something || (something = {}))

console.log(something); // {foo:123, bar:456}

```

Це часто використовується в JavaScript для того, щоб переконатися, що елементи не витікають в глобальний простір імен. З файловими модулями вам не потрібно хвилюватися про це, але шаблон все ще корисний для *логічної групування* кількох функцій. Тому TypeScript надає ключове слово `namespace` для групування цих елементів, наприклад:

```ts
namespace Utility {
    export function log(msg) {
        console.log(msg);
    }
    export function error(msg) {
        console.error(msg);
    }
}

// використання
Utility.log('Call me');
Utility.error('maybe!');
```

Ключове слово `namespace` генерує той самий JavaScript, який ми бачили раніше:

```ts
(function (Utility) {

// Додавання елементів до Utility

})(Utility || (Utility = {}));
```

Одним з важливих моментів є те, що простори імен можуть бути вкладені, тому можна використовувати `namespace Utility.Messaging`, щоб вкласти простір імен `Messaging` під `Utility`.

Для більшості проектів ми рекомендуємо використовувати зовнішні модулі та використовувати `namespace` для швидких демонстрацій та перенесення старого коду JavaScript.