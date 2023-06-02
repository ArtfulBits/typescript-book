## Generics

Ключовою мотивацією для generics є документування значущих залежностей типів між членами. Члени можуть бути:

* Члени екземпляру класу
* Методи класу
* Аргументи функції
* Повернення значення функції

## Мотивація та приклади

Розгляньте просту реалізацію структури даних `Queue` (перший вход, перший вихід). Простий приклад на TypeScript / JavaScript виглядає так:

```ts
class Queue {
  private data = [];
  push(item) { this.data.push(item); }
  pop() { return this.data.shift(); }
}
```

Одна з проблем з цією реалізацією полягає в тому, що вона дозволяє людям додавати *будь-що* до черги, і коли вони її видаляють - це може бути *будь-що*. Це показано нижче, де хтось може додати `string` до черги, тоді як використання фактично передбачає, що тільки `numbers` були додані:

```ts
class Queue {
  private data = [];
  push(item) { this.data.push(item); }
  pop() { return this.data.shift(); }
}

const queue = new Queue();
queue.push(0);
queue.push("1"); // Oops a mistake

// a developer walks into a bar
console.log(queue.pop().toPrecision(1));
console.log(queue.pop().toPrecision(1)); // RUNTIME ERROR
```

Один з варіантів (і насправді єдиний в мовах, які не підтримують generics) - створити *спеціальні* класи саме для цих обмежень. Наприклад, швидкий і нечистий черга чисел:

```ts
class QueueNumber extends Queue {
  push(item: number) { super.push(item); }
  pop(): number { return this.data.shift(); }
}

const queue = new QueueNumber();
queue.push(0);
queue.push("1"); // ERROR : cannot push a string. Only numbers allowed

// ^ if that error is fixed the rest would be fine too
```

Звичайно, це може швидко стати болісним, наприклад, якщо ви хочете чергу рядків, вам доведеться зробити всі зусилля знову. Те, що ви дійсно хочете, - це спосіб сказати, що будь-який тип речей, які отримують *pushed*, повинен бути таким самим для того, що отримує *popped*. Це легко зробити за допомогою *generic* параметра (у цьому випадку на рівні класу):

```ts
/** A class definition with a generic parameter */
class Queue<T> {
  private data = [];
  push(item: T) { this.data.push(item); }
  pop(): T | undefined { return this.data.shift(); }
}

/** Again sample usage */
const queue = new Queue<number>();
queue.push(0);
queue.push("1"); // ERROR : cannot push a string. Only numbers allowed

// ^ if that error is fixed the rest would be fine too
```

Інший приклад, який ми вже бачили, - це функція *reverse*, тут обмеження між тим, що передається в функцію, і тим, що функція повертає:

```ts
function reverse<T>(items: T[]): T[] {
    var toreturn = [];
    for (let i = items.length - 1; i >= 0; i--) {
        toreturn.push(items[i]);
    }
    return toreturn;
}

var sample = [1, 2, 3];
var reversed = reverse(sample);
console.log(reversed); // 3,2,1

// Safety!
reversed[0] = '1';     // Error!
reversed = ['1', '2']; // Error!

reversed[0] = 1;       // Okay
reversed = [1, 2];     // Okay
```

У цьому розділі ви побачили приклади generics, що визначаються *на рівні класу* та на *рівні функції*. Одним незначним доповненням, яке варто згадати, є те, що ви можете створювати generics тільки для функції-члена. Як іграшковий приклад розгляньте наступне, де ми переміщуємо функцію `reverse` в клас `Utility`:

```ts
class Utility {
  reverse<T>(items: T[]): T[] {
      var toreturn = [];
      for (let i = items.length - 1; i >= 0; i--) {
          toreturn.push(items[i]);
      }
      return toreturn;
  }
}
```

> ПОРАДА: Ви можете називати generic параметр будь-яким ім'ям. Звичайно використовувати `T`, `U` або `V`, коли у вас є прості generics. Якщо у вас є більше одного generic аргументу, спробуйте використовувати значущі назви, такі як `TKey` та `TValue`. Конвенція полягає в тому, що generics також називають *шаблонами* в інших мовах, таких як C ++.

### Шаблон проектування: Зручний generic

Розгляньте функцію:

```ts
declare function parse<T>(name: string): T;
```

У цьому випадку ви можете побачити, що тип `T` використовується тільки в одному місці. Таким чином, немає обмеження *між* членами. Це еквівалентно твердженню типу з точки зору безпеки типів:

```ts
declare function parse(name: string): any;

const something = parse('something') as TypeOfSomething;
```

Generics, які використовуються *лише один раз*, не кращі за твердження з точки зору безпеки типів. Зате вони забезпечують *зручність* вашого API.

Більш очевидний приклад - це функція, яка завантажує відповідь json. Вона повертає обіцянку *будь-якого типу, який ви передаєте*:

```ts
const getJSON = <T>(config: {
    url: string,
    headers?: { [key: string]: string },
  }): Promise<T> => {
    const fetchConfig = ({
      method: 'GET',
      'Accept': 'application/json',
      'Content-Type': 'application/json',
      ...(config.headers || {})
    });
    return fetch(config.url, fetchConfig)
      .then<T>(response => response.json());
  }
```

Зверніть увагу, що вам все ще потрібно явно анотувати те, що ви хочете, але сигнатура `getJSON<T>` `(config) => Promise<T>` зберігає вам кілька клавіш (вам не потрібно анотувати тип повернення `loadUsers`, оскільки його можна вивести):

```ts
type LoadUsersResponse = {
  users: {
    name: string;
    email: string;
  }[];  // array of user objects
}
function loadUsers() {
  return getJSON<LoadUsersResponse>({ url: 'https://example.com/users' });
}
```

Також `Promise<T>` як повернення значення, безумовно, краще за альтернативи, такі як `Promise<any>`.

Інший приклад полягає в тому, що generic використовується тільки як аргумент:

```ts
declare function send<T>(arg: T): void;
```

Тут generic `T` може бути використаний для анотації типу, який ви хочете, щоб аргумент відповідав, наприклад:

```ts
send<Something>({
  x:123,
  // Also you get autocomplete  
}); // Will TSError if `x:123` does not match the structure expected for Something

```