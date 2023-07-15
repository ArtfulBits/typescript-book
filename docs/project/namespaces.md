## Namespaces
Простори імен надають вам зручний синтаксис навколо загального шаблону, який використовується в JavaScript:

```ts
(function(something) {

    something.foo = 123;

})(something || (something = {}))
```

В основному `something || (something = {})` дозволяє анонімній функції `function(something) {}` *add stuff to an existing object* (`something`) або *start a new object then add stuff to that object* (`|| (something = {})`). Це означає, що ви можете мати два таких блоки, розділені деякою межею виконання:
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

Це зазвичай використовується в JavaScript, щоб переконатися, що матеріал не просочується в глобальний простір імен. З модулями на основі файлів вам не потрібно турбуватися про це, але шаблон все ще корисний для *logical grouping* купи функцій. Тому TypeScript надає ключове слово `namespace` для групування, наприклад:

```ts
namespace Utility {
    export function log(msg) {
        console.log(msg);
    }
    export function error(msg) {
        console.error(msg);
    }
}

// usage
Utility.log('Call me');
Utility.error('maybe!');
```

Ключове слово `namespace` генерує той самий JavaScript, який ми бачили раніше:

```ts
(function (Utility) {

// Add stuff to Utility

})(Utility || (Utility = {}));
```

Варто зауважити, що простори імен можуть бути вкладеними, тому ви можете робити такі речі, як `namespace Utility.Messaging`, щоб вкладати простір імен `Messaging` у `Utility`.

Для більшості проектів ми рекомендуємо використовувати зовнішні модулі та `namespace` для швидких демонстрацій і перенесення старого коду JavaScript.
