# Never
> [Youtube: Video lesson on the never type](https://www.youtube.com/watch?v=aldIFYWu6xc)

> [Egghead: Video lesson on the never type](https://egghead.io/lessons/typescript-use-the-never-type-to-avoid-code-with-dead-ends-using-typescript)

Дизайн мови програмування дійсно має концепцію *низького* типу, що є **natural** результатом, щойно ви виконуєте *code flow analysis*, тому йому потрібно надійно представляти те, що може ніколи не статися.

Тип `never` використовується в TypeScript для позначення цього *bottom* Випадки, коли це відбувається природним шляхом:

* Функція ніколи не повертає керування (наприклад, якщо тіло функції має `while(true){}`)
* Функція завжди видає помилку (наприклад, у `function foo(){throw new Error('Not Implemented')}` тип повернення `foo` - `never`)

Звичайно, ви також можете використовувати цю анотацію самостійно

```ts
let foo: never; // Okay
```

Однак *тільки `never` може бути значення іншого never*.

```ts
let foo: never = 123; // Error: Tип number не зпівпадає з типом never

// Okay ця функція повертає `never`
let bar: never = (() => { throw new Error(`Throw my hands in the air like I just don't care`) })();
```

Чудово Тепер давайте просто перейдемо до його ключового випадку використання :)

# Use case: Exhaustive Checks

Ви можете викликати функції never у контексті never.

```ts
function foo(x: string | number): boolean {
  if (typeof x === "string") {
    return true;
  } else if (typeof x === "number") {
    return false;
  }

  // Без типу never ми б мали помилку:
   // - Не всі шляхи коду повертають значення (суворі перевірки на нуль)
   // - Або виявлено недоступний код
   // Але оскільки TypeScript розуміє, що функція `fail` повертає `never`
   // Це може дозволити вам викликати його, оскільки ви, можливо, використовуєте його для безпеки під час виконання / вичерпних перевірок.
  return fail("Unexhaustive!");
}

function fail(message: string): never { throw new Error(message); }
```

А оскільки `never` можна призначити лише іншому `never`, ви також можете використовувати його для *compile time* вичерпних перевірок. Це описано в розділі [*discriminated union* section](./discriminated-unions.md).

# Confusion with `void`

Як тільки хтось скаже вам, що `never` повертається, коли функція ніколи не виходить витончено, ви інтуїтивно хочете думати про це як про те саме, що `void`. Однак `void` є істотою. `never` є завжди неіснуючим.

Функція, яка *повертає* нічого, повертає Unit `void`. Однак функція, *яка ніколи не повертає* (або завжди викидає), повертає `never`. `void` — це те, що можна призначити (без `strictNullChecking`), але `never` не можна *ніколи* призначити нічого іншого, крім `never`.

# Type inference in never returning functions

Для оголошень функцій TypeScript за замовчуванням визначає `void`, як показано нижче:

```ts
// Виявлений тип повернення: void
function failDeclaration(message: string) {
  throw new Error(message);
}

// Виявлений тип повернення: never
const failExpression = function(message: string) {
  throw new Error(message);
};
```

Звичайно, ви можете виправити це за допомогою явної анотації:

```ts
function failDeclaration(message: string): never {
  throw new Error(message);
}
```

Основною причиною є сумісність зворотного слова з реальним кодом JavaScript:

```ts
class Base {
    overrideMe() {
        throw new Error("You forgot to override me!");
    }
}

class Derived extends Base {
    overrideMe() {
        // Code that actually returns here
    }
}
```

If `Base.overrideMe` . 

> Реальний TypeScript може подолати це за допомогою `abstract` функцій, але цей висновок підтримується для сумісності.
<!--
PR: https://github.com/Microsoft/TypeScript/pull/8652
Issue : https://github.com/Microsoft/TypeScript/issues/3076
Concept : https://en.wikipedia.org/wiki/Bottom_type
-->
