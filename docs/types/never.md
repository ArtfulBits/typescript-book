# Ніколи
> [Youtube: Відеоурок про тип "ніколи"](https://www.youtube.com/watch?v=aldIFYWu6xc)

> [Egghead: Відеоурок про тип "ніколи"](https://egghead.io/lessons/typescript-use-the-never-type-to-avoid-code-with-dead-ends-using-typescript)

Проектування мов програмування має концепцію *нижнього* типу, який є **природним** результатом, як тільки ви робите *аналіз потоку коду*. TypeScript робить *аналіз потоку коду* (😎), тому йому потрібно надійно представляти речі, які можуть ніколи не статися.

В TypeScript тип `never` використовується для позначення цього *нижнього* типу. Випадки, коли він виникає природно:

* Функція ніколи не повертається (наприклад, якщо тіло функції містить `while(true){}`)
* Функція завжди кидає виключення (наприклад, у функції `function foo(){throw new Error('Not Implemented')}` тип повернення `foo` є `never`)

Звичайно, ви також можете використовувати цю анотацію самостійно

```ts
let foo: never; // Ок
```

Однак *тільки `never` може бути призначений для іншого `never`*. наприклад:

```ts
let foo: never = 123; // Помилка: Тип number не може бути призначений для never

// Ок, оскільки тип повернення функції є `never`
let bar: never = (() => { throw new Error(`Throw my hands in the air like I just don't care`) })();
```

Чудово. Тепер давайте перейдемо до його ключового використання :)

# Використання: Вичерпні перевірки

Ви можете викликати функції "ніколи" в контексті "ніколи".

```ts
function foo(x: string | number): boolean {
  if (typeof x === "string") {
    return true;
  } else if (typeof x === "number") {
    return false;
  }

  // Без типу "ніколи" ми отримаємо помилку:
  // - Не всі шляхи коду повертають значення (строга перевірка на null)
  // - Або виявлено недосяжний код
  // Але оскільки TypeScript розуміє, що функція "fail" повертає "never"
  // Ви можете викликати її, оскільки ви можете використовувати її для забезпечення безпеки виконання / вичерпних перевірок.
  return fail("Unexhaustive!");
}

function fail(message: string): never { throw new Error(message); }
```

І оскільки `never` може бути призначений тільки для іншого `never`, ви можете використовувати його для *перевірок на етапі компіляції*. Це описано в розділі [*об'єднання з дискримінантом*](./discriminated-unions.md).

# Плутанина з `void`

Як тільки хтось каже вам, що `never` повертається, коли функція ніколи не виходить з блоку, ви інтуїтивно хочете подумати про це як про те саме, що й `void`. Однак `void` є одиницею. `never` є фальсумом.

Функція, яка *не повертає* нічого, повертає одиницю `void`. Однак функція, *яка ніколи не повертається* (або завжди кидає виключення), повертає `never`. `void` - це щось, що можна призначити (без `strictNullChecking`), але `never` може *ніколи* не бути призначений для чогось іншого, крім `never`.

# Інференція типу в функціях, які ніколи не повертають значення

Для оголошень функцій TypeScript за замовчуванням виводить тип `void`, як показано нижче:

```ts
// Виведений тип повернення: void
function failDeclaration(message: string) {
  throw new Error(message);
}

// Виведений тип повернення: never
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

Основна причина полягає в зворотній сумісності з реальними JavaScript-кодом:

```ts
class Base {
    overrideMe() {
        throw new Error("You forgot to override me!");
    }
}

class Derived extends Base {
    overrideMe() {
        // Код, який насправді повертається тут
    }
}
```

Якщо `Base.overrideMe`.

> У реальному світі TypeScript може подолати це за допомогою абстрактних функцій, але ця інференція зберігається для сумісності. 

<!--
PR: https://github.com/Microsoft/TypeScript/pull/8652
Issue : https://github.com/Microsoft/TypeScript/issues/3076
Concept : https://en.wikipedia.org/wiki/Bottom_type
-->