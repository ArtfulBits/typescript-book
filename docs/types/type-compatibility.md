* [Type Compatibility](#type-compatibility)
* [Soundness](#soundness)
* [Structural](#structural)
* [Generics](#generics)
* [Variance](#variance)
* [Functions](#functions)
  * [Return Type](#return-type)
  * [Number of arguments](#number-of-arguments)
  * [Optional and rest parameters](#optional-and-rest-parameters)
  * [Types of arguments](#types-of-arguments)
* [Enums](#enums)
* [Classes](#classes)
* [Generics](#generics)
* [FootNote: Invariance](#footnote-invariance)

## Type Compatibility

Сумісність типу (як ми обговорюємо тут) визначає, чи можна призначити один предмет іншому. наприклад `string` i `number` несумісні:

```ts
let str: string = "Hello";
let num: number = 123;

str = num; // ERROR: `number` is not assignable to `string`
num = str; // ERROR: `string` is not assignable to `number`
```

## Soundness

Система типів TypeScript розроблена так, щоб бути зручною та допускати *unsound* поведінку, напр. будь-що можна призначити будь-якому, що означає, що компілятор повинен дозволити вам робити все, що завгодно:

```ts
let foo: any = 123;
foo = "Hello";

// Later
foo.toPrecision(3); // Allowed as you typed it as `any`
```

## Structural

Об'єкти TypeScript є структурно типізованими. Це означає, що *іменуванняs* не мають значення, якщо структури збігаються

```ts
interface Point {
    x: number,
    y: number
}

class Point2D {
    constructor(public x:number, public y:number){}
}

let p: Point;
// OK, because of structural typing
p = new Point2D(1,2);
```

Це дозволяє вам створювати об’єкти на льоту (як ви це робите у vanilla JS) і залишатися в безпеці, коли це можна зробити.

Також *більше* даних вважається правильним

```ts
interface Point2D {
    x: number;
    y: number;
}
interface Point3D {
    x: number;
    y: number;
    z: number;
}
var point2D: Point2D = { x: 0, y: 10 }
var point3D: Point3D = { x: 0, y: 10, z: 20 }
function iTakePoint2D(point: Point2D) { /* do something */ }

iTakePoint2D(point2D); // exact match okay
iTakePoint2D(point3D); // extra information okay
iTakePoint2D({ x: 0 }); // Error: missing information `y`
```

## Variance

Дисперсія — це легке для розуміння та важливе поняття для аналізу сумісності типів.

Для простих типів `Base` і `Child`, якщо `Child` є дочірнім типом `Base`, тоді екземпляри `Child` можна призначити змінній типу `Base`.

> Це поліморфізм 101

Сумісність типів складних типів, що складаються з таких типів `Base` і `Child`, залежить від того, де `Base` і `Child` у подібних сценаріях керуються *variance*.

* Коваріант: (співпраця, він же спільний) тільки в *спільному напрямку*
* Контраваріант: (контра, він же негативний) тільки в*oпротилежний напрямок*
* Біваріант : (bi або обидва) і co, і contra.
* Invariant : якщо типи не зовсім однакові, вони несумісні.* 

> Примітка: для повністю надійної системи за наявності змінних даних, таких як JavaScript, `invariant` є єдиним дійсним варіантом. Але, як уже згадувалося, *зручність* змушує нас робити необґрунтований вибір.

## Functions

Порівнюючи дві функції, слід враховувати кілька тонких моментів.

### Return Type

`covariant`: тип повернення має містити принаймні достатньо даних.

```ts
/** Type Hierarchy */
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

/** Two sample functions */
let iMakePoint2D = (): Point2D => ({ x: 0, y: 0 });
let iMakePoint3D = (): Point3D => ({ x: 0, y: 0, z: 0 });

/** Assignment */
iMakePoint2D = iMakePoint3D; // Okay
iMakePoint3D = iMakePoint2D; // ERROR: Point2D is not assignable to Point3D
```

### Number of arguments

Менша кількість аргументів — це добре (тобто функції можуть ігнорувати додаткові параметри). Адже вам гарантовано викличете їх із як мінімум достатніми аргументами.

```ts
let iTakeSomethingAndPassItAnErr
    = (x: (err: Error, data: any) => void) => { /* do something */ };

iTakeSomethingAndPassItAnErr(() => null) // Okay
iTakeSomethingAndPassItAnErr((err) => null) // Okay
iTakeSomethingAndPassItAnErr((err, data) => null) // Okay

// ERROR: Argument of type '(err: any, data: any, more: any) => null' is not assignable to parameter of type '(err: Error, data: any) => void'.
iTakeSomethingAndPassItAnErr((err, data, more) => null);
```

### Optional and Rest Parameters

Необов’язкові (попередньо визначена кількість) і параметри Rest (будь-яка кількість аргументів) сумісні, знову ж для зручності.

```ts
let foo = (x:number, y: number) => { /* do something */ }
let bar = (x?:number, y?: number) => { /* do something */ }
let bas = (...args: number[]) => { /* do something */ }

foo = bar = bas;
bas = bar = foo;
```

> Примітка: необов’язковий (у нашому прикладі `bar`) і необов’язковий (у нашому прикладі `foo`) сумісні, лише якщо strictNullChecks має значення false.

### Types of arguments

`bivariant` : це розроблено для підтримки типових сценаріїв обробки подій

```ts
/** Event Hierarchy */
interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

/** Sample event listener */
enum EventType { Mouse, Keyboard }
function addEventListener(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// Unsound, but useful and common. Works as function argument comparison is bivariant
addEventListener(EventType.Mouse, (e: MouseEvent) => console.log(e.x + "," + e.y));

// Undesirable alternatives in presence of soundness
addEventListener(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + "," + (<MouseEvent>e).y));
addEventListener(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + "," + e.y)));

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
addEventListener(EventType.Mouse, (e: number) => console.log(e));
```

Крім того, `Array<Child>` можна призначити `Array<Base>` (коваріація), оскільки функції сумісні. Коваріація масиву вимагає, щоб усі функції `Array<Child>` можна було призначити `Array<Base>`, наприклад. `push(t:Child)` можна призначити `push(t:Base)`, що стало можливим завдяки біваріантності аргументів функції.

**Це може збити з пантелику людей, які знають інші мови**, які очікували б наступного помилки, але не в TypeScript:

```ts
/** Type Hierarchy */
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

/** Two sample functions */
let iTakePoint2D = (point: Point2D) => { /* do something */ }
let iTakePoint3D = (point: Point3D) => { /* do something */ }

iTakePoint3D = iTakePoint2D; // Okay : Reasonable
iTakePoint2D = iTakePoint3D; // Okay : WHAT
```

## Enums

* Enum сумісні з числами, а числа сумісні з enum.

```ts
enum Status { Ready, Waiting };

let status = Status.Ready;
let num = 0;

status = num; // OKAY
num = status; // OKAY
```

** Значення Enum з різних типів enum вважаються несумісними. Це робить переліки придатними для використання *номінально* (на відміну від структурних)

```ts
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
let color = Color.Red;

status = color; // ERROR
```

## Classes

* Порівнюються лише члени екземпляра та методи.*constructors* і *statics* не грають ролі.

```ts
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { /** do something */ }
}

class Size {
    feet: number;
    constructor(meters: number) { /** do something */ }
}

let a: Animal;
let s: Size;

a = s;  // OK
s = a;  // OK
```

* `private` та `protected` члени *повинні походити з того самого класу*. Такі члени по суті роблять клас *nominal*.

```ts
/** A class hierarchy */
class Animal { protected feet: number; }
class Cat extends Animal { }

let animal: Animal;
let cat: Cat;

animal = cat; // OKAY
cat = animal; // OKAY

/** Looks just like Animal */
class Size { protected feet: number; }

let size: Size;

animal = size; // ERROR
size = animal; // ERROR
```

## Generics

Оскільки TypeScript має структурну систему типів, параметри типу впливають на сумісність лише тоді, коли вони використовуються членом. Наприклад, у наступному `T` не впливає на сумісність:

```ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // okay, y matches structure of x
```

Однак, якщо використовується `T`, він відіграватиме роль у сумісності на основі свого *екземпляру*, як показано нижче:

```ts
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // error, x and y are not compatible
```

У випадках, коли загальні аргументи не були *інстанцировані*, вони замінюються на `any` перед перевіркою сумісності:

```ts
let identity = function<T>(x: T): T {
    // ...
}

let reverse = function<U>(y: U): U {
    // ...
}

identity = reverse; // Добре, тому що ((x: any)=>any відповідає (y: any)=>any
```

Універсали, що включають класи, відповідають відповідній сумісності класів, як згадувалося раніше

```ts
class List<T> {
  add(val: T) { }
}

class Animal { name: string; }
class Cat extends Animal { meow() { } }

const animals = new List<Animal>();
animals.add(new Animal()); // Okay 
animals.add(new Cat()); // Okay 

const cats = new List<Cat>();
cats.add(new Animal()); // Error 
cats.add(new Cat()); // Okay
```

## FootNote: Invariance

Ми сказали, що інваріантність — це єдиний правильний варіант. Ось приклад, коли дисперсія `contra` і `co` небезпечна для масивів.

```ts
/** Hierarchy */
class Animal { constructor(public name: string){} }
class Cat extends Animal { meow() { } }

/** An item of each */
var animal = new Animal("animal");
var cat = new Cat("cat");

/**
 * Demo : polymorphism 101
 * Animal <= Cat
 */
animal = cat; // Okay
cat = animal; // ERROR: cat extends animal

/** Array of each to demonstrate variance */
let animalArr: Animal[] = [animal];
let catArr: Cat[] = [cat];

/**
 * Obviously Bad : Contravariance
 * Animal <= Cat
 * Animal[] >= Cat[]
 */
catArr = animalArr; // Okay if contravariant
catArr[0].meow(); // Дозволено, але BANG 🔫 під час виконання


/**
 * Also Bad : covariance
 * Animal <= Cat
 * Animal[] <= Cat[]
 */
animalArr = catArr; // Okay if covariant
animalArr.push(new Animal('another animal')); // Just pushed an animal into catArr!
catArr.forEach(c => c.meow()); // Дозволено, але BANG 🔫 під час виконання
```
