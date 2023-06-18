## Generics

Основна мотивація для дженериків — документувати значущі залежності типів між членами. Членами можуть бути:

* Члени примірника класу
* Методи класу
* аргументи функції
* значення, що повертається функцією

## Motivation and samples

Розглянемо просту реалізацію структури даних `Queue` (першим прийшов, першим вийшов). Приклад у TypeScript / JavaScript виглядає так:

```ts
class Queue {
  private data = [];
  push(item) { this.data.push(item); }
  pop() { return this.data.shift(); }
}
```

Одна проблема з цією реалізацією полягає в тому, що вона дозволяє людям додавати *будь-що* до черги, а коли вони відкривають її, це може бути *нічого*. Це показано нижче, де хтось може вставити `string` в чергу, тоді як використання фактично передбачає, що було вставлено лише `numbers`:

```ts
class Queue {
  private data = [];
  push(item) { this.data.push(item); }
  pop() { return this.data.shift(); }
}

const queue = new Queue();
queue.push(0);
queue.push("1"); // Помилка

// a developer walks into a bar
console.log(queue.pop().toPrecision(1));
console.log(queue.pop().toPrecision(1)); // RUNTIME ERROR
```

Одне з рішень (і фактично єдине в мовах, які не підтримують універсали(generics)) полягає в тому, щоб створити *special* класи тільки для цих обмежень. Наприклад швидка черга номерів:

```ts
class QueueNumber extends Queue {
  push(item: number) { super.push(item); }
  pop(): number { return this.data.shift(); }
}

const queue = new QueueNumber();
queue.push(0);
queue.push("1"); // ERROR : не можно додати string. Лише numbers дозволени

// ^ якщо цю помилку буде виправлено, решта теж буде добре
```
Звичайно, це може швидко стати болісним, напр. якщо вам потрібна черга рядків, вам доведеться знову пройти через усі ці зусилля. Те, що вам дійсно потрібно, це спосіб сказати, що незалежно від типу матеріалу, який *додаеться* він повинен бути таким самим, як *видається*. Це легко зробити за допомогою параметра *generic* (у цьому випадку на рівні класу):

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
queue.push("1"); //  // ERROR : не можно додати string. Лише numbers дозволени

// ^ якщо цю помилку буде виправлено, решта теж буде добре
```

Ще один приклад, який ми вже бачили, це функція *reverse*, тут обмеження між тим, що передається у функцію, і тим, що функція повертає:

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

У цьому розділі ви бачили приклади генериків, визначених *at class level* та на *function level*. Одне невелике доповнення, яке варто згадати, полягає в тому, що ви можете створювати генерики лише для функції-члена. Як приклад розглянемо наступне, де ми переміщуємо функцію `reverse` в клас `Utility`:

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

> ПОРАДА: Ви можете називати загальний параметр як завгодно. Традиційно використовують `T`, `U` або `V`, коли у вас є прості генерики. Якщо у вас є більше ніж один загальний аргумент, спробуйте використовувати значущі імена, як-от `TKey` і `TValue`. Конвенція передбачає префікс `T`, оскільки генерики також називаються *templates* в інших мовах, наприклад C++.


### Design Pattern: Convenience generic

Розглянемо функцію:

```ts
declare function parse<T>(name: string): T;
```

У цьому випадку ви можете побачити, що тип `T` використовується лише в одному місці. Тому немає обмежень *between* членами. Це еквівалентно твердженню типу з точки зору безпеки типу:

```ts
declare function parse(name: string): any;

const something = parse('something') as TypeOfSomething;
```

Дженерики, використані *only once* не кращі за твердження з точки зору безпеки типу. Тим не менш, вони забезпечують *convenience* для вашого API.

Більш очевидним прикладом є функція, яка завантажує відповідь json. Він повертає обіцянку *whatever type you pass in*:
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

Зауважте, що вам все одно потрібно явно анотувати те, що ви хочете, але підпис `getJSON<T>` `(config) => Promise<T>` заощаджує вам кілька натискань клавіш (вам не потрібно коментувати тип повернення `loadUsers`, як це можна зробити висновок):

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

Крім того, `Promise<T>` як значення, що повертається, однозначно краще, ніж альтернативи, такі як `Promise<any>`.

Іншим прикладом є те, що загальне слово використовується лише як аргумент:

```ts
declare function send<T>(arg: T): void;
```

Тут загальний `T` можна використовувати для позначення типу, якому ви хочете, щоб аргумент відповідав, наприклад.

```ts
send<Something>({
  x:123,
 // Також ви отримуєте автозаповнення
}); // Буде TSError, якщо `x:123` не відповідає структурі, очікуваній для Щось

```
