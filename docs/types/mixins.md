# Mixins

Класи TypeScript (і JavaScript) підтримують суворе єдине успадкування. Отже, ви *не зможете* робити:

```ts
class User extends Tagged, Timestamped { // ERROR : no multiple inheritance
}
```

ший спосіб побудови класів із багаторазово використовуваних компонентів — побудувати їх шляхом об’єднання простіших часткових класів, які називаються міксинами.

Ідея проста: замість того, щоб *class A extending class B* to get its functionality,щоб отримати його функціональність, *function B takes class A* і повертає новий клас із цією доданою функціональністю. Функція `B` є міксином.

> [Міксин — це] функція, яка
>
> 1. бере конструктор,
> 1. створює клас, який розширює цей конструктор новою функціональністю
> 1. повертає новий клас

Повний приклад

```ts
// Ніобхідно для всіх міксінов
type Constructor<T = {}> = new (...args: any[]) => T;

////////////////////
// Приклад mixins
////////////////////

// Мixin який додає властивість
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}

//  Мixin який додає властивість та методи
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActivated = false;

    activate() {
      this.isActivated = true;
    }

    deactivate() {
      this.isActivated = false;
    }
  };
}

////////////////////
// використання
////////////////////

// Простий class
class User {
  name = '';
}

// User that is Timestamped
const TimestampedUser = Timestamped(User);

// User that is Timestamped and Activatable
const TimestampedActivatableUser = Timestamped(Activatable(User));

////////////////////
// Использование расширеного classes
////////////////////

const timestampedUserExample = new TimestampedUser();
console.log(timestampedUserExample.timestamp);

const timestampedActivatableUserExample = new TimestampedActivatableUser();
console.log(timestampedActivatableUserExample.timestamp);
console.log(timestampedActivatableUserExample.isActivated);

```

Розкладемо цей приклад.

## Take a constructor

Міксини беруть клас і розширюють його новими функціями. Отже, нам потрібно визначити, що таке *constructor*. Легко, як:

```ts
// Needed for all mixins
type Constructor<T = {}> = new (...args: any[]) => T;
```

## Extend the class and return it

Досить легко:

```ts
// A mixin that adds a property
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}
```

І це все🌹
