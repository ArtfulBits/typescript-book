# Міксіни

Класи TypeScript (і JavaScript) підтримують строгу одинарну спадковість. Тому ви *не можете* зробити наступне:

```ts
class User extends Tagged, Timestamped { // ПОМИЛКА: немає множинної спадковості
}
```

Інший спосіб створення класів з перевикористовуваних компонентів полягає в їх створенні шляхом поєднання простіших часткових класів, які називаються міксінами.

Ідея проста: замість того, щоб *клас A розширює клас B* для отримання його функціональності, *функція B бере клас A* і повертає новий клас з доданою функціональністю. Функція `B` є міксіном.

> [Міксін - це] функція, яка
>
> 1. приймає конструктор,
> 1. створює клас, який розширює цей конструктор з новою функціональністю,
> 1. повертає новий клас.

Повний приклад

```ts
// Потрібно для всіх міксінів
type Constructor<T = {}> = new (...args: any[]) => T;

////////////////////
// Приклад міксінів
////////////////////

// Міксін, який додає властивість
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}

// міксін, який додає властивість та методи
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
// Використання для створення класів
////////////////////

// Простий клас
class User {
  name = '';
}

// Користувач, який має Timestamped
const TimestampedUser = Timestamped(User);

// Користувач, який має Timestamped та Activatable
const TimestampedActivatableUser = Timestamped(Activatable(User));

////////////////////
// Використання створених класів
////////////////////

const timestampedUserExample = new TimestampedUser();
console.log(timestampedUserExample.timestamp);

const timestampedActivatableUserExample = new TimestampedActivatableUser();
console.log(timestampedActivatableUserExample.timestamp);
console.log(timestampedActivatableUserExample.isActivated);

```

Розберемо цей приклад.

## Взяти конструктор

Міксіни беруть клас і розширюють його новою функціональністю. Тому нам потрібно визначити, що таке *конструктор*. Це просто:

```ts
// Потрібно для всіх міксінів
type Constructor<T = {}> = new (...args: any[]) => T;
```

## Розширити клас і повернути його

Досить просто:

```ts
// Міксін, який додає властивість
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}
```

І все 🌹