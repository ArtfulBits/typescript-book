## Interfaces

Інтерфейси мають *zero* вплив на час виконання JS. В інтерфейсах TypeScript є багато можливостей для оголошення структури змінних.

Наступні дві еквівалентні декларації, перша використовує*inline annotation*, друга використовує *interface*:

```ts
// Sample A
declare var myPoint: { x: number; y: number; };

// Sample B
interface Point {
    x: number; y: number;
}
declare var myPoint: Point;
```

Однак принадність *Sample B* полягає в тому, що якщо хтось створює бібліотеку, яка будується на бібліотеці `myPoint` для додавання нових учасників, вони можуть легко додати до існуючої декларації `myPoint`:

```ts
// Lib a.d.ts
interface Point {
    x: number; y: number;
}
declare var myPoint: Point;

// Lib b.d.ts
interface Point {
    z: number;
}

// Your code
var myPoint.z; // Allowed!
```

Це тому, що **interfaces in TypeScript are open ended**. Це життєво важливий принцип TypeScript, який дозволяє вам імітувати розширюваність JavaScript за допомогою *interfaces*.


## Classes can implement interfaces

Якщо ви хочете використовувати *classes* які повинні відповідати структурі об’єкта, яку хтось оголосив для вас в `interface` ви можете використати ключове слово `implements` для забезпечення сумісності:

```ts
interface Point {
    x: number; y: number;
}

class MyPoint implements Point {
    x: number; y: number; // Same as Point
}
```

По суті, за наявності цих `implements`, будь-які зміни в цьому зовнішньому інтерфейсі `Point` призведуть до помилки компіляції у вашій кодовій базі, тому ви можете легко підтримувати його в синхронізації:

```ts
interface Point {
    x: number; y: number;
    z: number; // New member
}

class MyPoint implements Point { // ERROR : missing member `z`
    x: number; y: number;
}
```

Зауважте, що `implements` обмежує структуру *instances* класу, тобто:

```ts
var foo: Point = new MyPoint();
```

І такі речі, як `foo: Point = MyPoint`, це не те саме.


## TIPs

### Not every interface is implementable easily

Інтерфейси призначені для оголошення *any arbitrarily crazy* структури, яка може бути присутня в JavaScript

Розглянемо наступний інтерфейс, де щось можна викликати за допомогою `new`:

```ts
interface Crazy {
    new (): {
        hello: number
    };
}
```

По суті, у вас буде щось на зразок:

```ts
class CrazyClass implements Crazy {
    constructor() {
        return { hello: 123 };
    }
}
// Because
const crazy = new CrazyClass(); // crazy would be {hello:123}
```

Ви можете *декларувати* всі божевільні JS з інтерфейсами та навіть безпечно використовувати їх із TypeScript. Це не означає, що ви можете використовувати класи TypeScript для їх реалізації.
