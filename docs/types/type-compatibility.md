* [Сумісність типів](#type-compatibility)
* [Звуковість](#soundness)
* [Структурна](#structural)
* [Загальні типи](#generics)
* [Варіантність](#variance)
* [Функції](#functions)
  * [Тип повернення](#return-type)
  * [Кількість аргументів](#number-of-arguments)
  * [Необов'язкові та залишкові параметри](#optional-and-rest-parameters)
  * [Типи аргументів](#types-of-arguments)
* [Перелічення](#enums)
* [Класи](#classes)
* [Загальні типи](#generics)
* [Примітка: Незмінність](#footnote-invariance)

## Сумісність типів

Сумісність типів (як ми тут обговорюємо) визначає, чи можна одне призначити іншому. Наприклад, `string` та `number` несумісні:

```ts
let str: string = "Hello";
let num: number = 123;

str = num; // ПОМИЛКА: `number` не може бути призначено `string`
num = str; // ПОМИЛКА: `string` не може бути призначено `number`
```

## Звуковість

Система типів TypeScript розроблена для зручності та дозволяє *незвукові* поведінки, наприклад, будь-що може бути призначено `any`, що означає, що ви кажете компілятору дозволити вам робити все, що ви хочете:

```ts
let foo: any = 123;
foo = "Hello";

// Пізніше
foo.toPrecision(3); // Дозволено, оскільки ви ввели його як `any`
```

## Структурна

Об'єкти TypeScript мають структурний тип. Це означає, що *назви* не мають значення, якщо структури співпадають

```ts
interface Point {
    x: number,
    y: number
}

class Point2D {
    constructor(public x:number, public y:number){}
}

let p: Point;
// Добре, через структурну типізацію
p = new Point2D(1,2);
```

Це дозволяє створювати об'єкти на льоту (як ви робите в vanilla JS) та все ще мати безпеку, коли це можна вивести.

Також *більше* даних вважається добре:

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
function iTakePoint2D(point: Point2D) { /* зробити щось */ }

iTakePoint2D(point2D); // точний збіг добре
iTakePoint2D(point3D); // додаткова інформація добре
iTakePoint2D({ x: 0 }); // Помилка: відсутня інформація `y`
```

## Варіантність

Варіантність - це легко зрозуміла та важлива концепція для аналізу сумісності типів.

Для простих типів `Base` та `Child`, якщо `Child` є дитиною `Base`, то екземпляри `Child` можуть бути призначені змінній типу `Base`.

> Це поліморфізм 101

У сумісності типів складних типів, що складаються з таких типів `Base` та `Child`, залежить від того, де `Base` та `Child` в подібних сценаріях визначається *варіантністю*.

* Коваріантний: (co, тобто спільний) тільки в *одному напрямку*
* Контраваріантний: (contra, тобто негативний) тільки в *протилежному напрямку*
* Біваріантний: (bi, тобто обидва) як co, так і contra.
* Незмінний: якщо типи не точно співпадають, то вони несумісні.

> Зауважте: для повністю звукової системи типів в присутності змінних даних, таких як JavaScript, *незмінність* є єдиним дійсним варіантом. Але, як зазначено, *зручність* змушує нас робити незвукові вибори.

## Функції

Є кілька тонких моментів, які потрібно враховувати при порівнянні двох функцій.

### Тип повернення

`коваріантний`: тип повернення повинен містити принаймні достатньо даних.

```ts
/** Ієрархія типів */
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

/** Дві зразки функцій */
let iMakePoint2D = (): Point2D => ({ x: 0, y: 0 });
let iMakePoint3D = (): Point3D => ({ x: 0, y: 0, z: 0 });

/** Призначення */
iMakePoint2D = iMakePoint3D; // Добре
iMakePoint3D = iMakePoint2D; // ПОМИЛКА: Point2D не може бути призначено Point3D
```

### Кількість аргументів

Менша кількість аргументів допустима (тобто функції можуть вибирати ігнорувати додаткові параметри). В кінці кінців, вам гарантовано, що вас викличуть з принаймні достатньою кількістю аргументів.

```ts
let iTakeSomethingAndPassItAnErr
    = (x: (err: Error, data: any) => void) => { /* зробити щось */ };

iTakeSomethingAndPassItAnErr(() => null) // Добре
iTakeSomethingAndPassItAnErr((err) => null) // Добре
iTakeSomethingAndPassItAnErr((err, data) => null) // Добре

// ПОМИЛКА: Аргумент типу '(err: any, data: any, more: any) => null' не може бути призначений параметру типу '(err: Error, data: any) => void'.
iTakeSomethingAndPassItAnErr((err, data, more) => null);
```

### Необов'язкові та залишкові параметри

Необов'язкові (заздалегідь визначена кількість) та залишкові параметри (будь-яка кількість аргументів) сумісні, знову ж таки, для зручності.

```ts
let foo = (x:number, y: number) => { /* зробити щось */ }
let bar = (x?:number, y?: number) => { /* зробити щось */ }
let bas = (...args: number[]) => { /* зробити щось */ }

foo = bar = bas;
bas = bar = foo;
```

> Зауважте: необов'язкові (у нашому прикладі `bar`) та необов'язкові (у нашому прикладі `foo`) сумісні тільки тоді, коли strictNullChecks дорівнює false.

### Типи аргументів

`біваріантний`: це призначено для

```uk
# Сумісність типів в TypeScript

TypeScript має структурну систему типів, що означає, що типи зі структурною сумісністю можуть бути привласнені один одному. Це може призвести до неочікуваної поведінки в деяких випадках. Нижче наведено кілька прикладів.

## Прості типи

* Прості типи повинні бути однакові, щоб бути сумісними.

```ts
let x: number;
let y: number | string;

x = y; // Помилка: типи несумісні
y = x; // OK
```

## Функції

* Функції повинні мати однакову кількість параметрів. Параметри також повинні мати сумісні типи.

```ts
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Помилка
```

* Тип повернення функції повинен мати тип, який може бути привласнений типу відповідного параметра.

```ts
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // Помилка
y = x; // OK, оскільки тип `y` містить тип `x`
```

## Об'єкти

* Об'єкти повинні мати однакову кількість властивостей. Властивості також повинні мати сумісні типи.

```ts
let x = {name: 'Alice'};
let y = {name: 'Alice', location: 'Seattle'};

x = y; // Помилка
y = x; // OK
```

* Якщо об'єкт містить приватні чи захищені властивості, то він повинен бути з того ж класу, щоб бути сумісним.

```ts
class Animal { feet: number; }
class Size { feet: number; }

let a: Animal;
let s: Size;

a = s; // OK
s = a; // OK, оскільки обидва мають однакові властивості

class Animal { protected feet: number; }
class Cat extends Animal { }

let animal: Animal;
let cat: Cat;

animal = cat; // OK
cat = animal; // OK, оскільки обидва мають однакові захищені властивості

class Size { protected feet: number; }

let size: Size;

animal = size; // Помилка
size = animal; // Помилка
```

## Enums

* Enums сумісні з числами, і числа сумісні з enums.

```ts
enum Status { Ready, Waiting };

let status = Status.Ready;
let num = 0;

status = num; // OK
num = status; // OK
```

* Значення enums з різних типів enums вважаються несумісними. Це робить enums використовуваними *номінально* (на відміну від структурних типів).

```ts
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
let color = Color.Red;

status = color; // Помилка
```

## Класи

* Порівнюються лише екземплярні властивості та методи. *Конструктори* та *статичні* члени не беруть участі в порівнянні.

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

* `private` та `protected` властивості *повинні походити з того ж класу*. Такі властивості в суті роблять клас *номінальним*.

```ts
/** Ієрархія класів */
class Animal { protected feet: number; }
class Cat extends Animal { }

let animal: Animal;
let cat: Cat;

animal = cat; // OK
cat = animal; // OK

/** Виглядає так само, як Animal */
class Size { protected feet: number; }

let size: Size;

animal = size; // Помилка
size = animal; // Помилка
```

## Generics

Оскільки TypeScript має структурну систему типів, параметри типу впливають на сумісність лише тоді, коли вони використовуються членом. Наприклад, у наступному прикладі `T` не впливає на сумісність:

```ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // OK, оскільки `y` відповідає структурі `x`
```

Однак, якщо `T` використовується, він впливає на сумісність на основі його *інстанціювання*, як показано нижче:

```ts
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // Помилка, `x` та `y` несумісні
```

У випадках, коли аргументи generics не були *інстанційовані*, вони замінюються на `any` перед перевіркою сумісності:

```ts
let identity = function<T>(x: T): T {
    // ...
}

let reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // OK, оскільки (x: any)=>any відповідає (y: any)=>any
```

Generics, що включають класи, відповідають відповідній сумісності класів, як зазначено вище. Наприклад:

```ts
class List<T> {
  add(val: T) { }
}

class Animal { name: string; }
class Cat extends Animal { meow() { } }

const animals = new List<Animal>();
animals.add(new Animal()); // OK 
animals.add(new Cat()); // OK 

const cats = new List<Cat>();
cats.add(new Animal()); // Помилка 
cats.add(new Cat()); // OK
```

## Примітка: інваріантність

Ми сказали, що інваріантність є єдиною безпечною опцією. Ось приклад, де як `contra`, так і `co` варіанти виявляються небезпечними для масивів.

```ts
/** Ієрархія */
class Animal { constructor(public name: string){} }
class Cat extends Animal { meow() { } }

/** Представники кожного */
var animal = new Animal("animal");
var cat = new Cat("cat");

/**
 * Демонстрація: поліморфізм 101
 * Animal <= Cat
 */
animal = cat; // OK
cat = animal; // Помилка: cat розширює animal

/** Масиви для демонстрації варіантності */
let animalArr: Animal[] = [animal];
let catArr: Cat[] = [cat];

/**
 * Очевидно погано: контраваріантність
 * Animal <= Cat
 * Animal[] >= Cat[]
 */
catArr = animalArr; // OK, якщо контраваріантний
catArr[0].meow(); // Дозволено, але BANG 🔫 під час виконання


/**
 * Також погано: коваріантність
 * Animal <= Cat
 * Animal[] <= Cat[]
 */
animalArr = catArr; // OK, якщо коваріантний
animalArr.push(new Animal('another animal')); // Щойно додали тварину до catArr!
catArr.forEach(c => c.meow()); // Дозволено, але BANG 🔫 під час виконання
```