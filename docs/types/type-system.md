# TypeScript Type System
Ми розглянули основні функції TypeScript Type System ще під час обговорення [Why TypeScript?](../why-typescript.md).
Нижче наведено кілька ключових висновків із цієї дискусії, які не потребують додаткових пояснень:

* Система типів у TypeScript розроблена як *optional*,так щоб *your JavaScript був TypeScript*.
* TypeScript не блокує *JavaScript emit* за наявності помилок типу, дозволяючи вам *поступово змінювати JS на TS*.

Тепер почнемо з *синтаксису* системи типів TypeScript. Таким чином ви можете негайно почати використовувати ці анотації у своєму коді та побачити переваги. Це підготує вас до більш глибокого занурення пізніше.

## Basic Annotations
Як згадувалося раніше, типи анотуються за допомогою синтаксису `:TypeAnnotation`. Усе, що є в просторі оголошення типу, можна використовувати як анотацію типу.

У наступному прикладі демонструються анотації типів для змінних, параметрів функції та значень, що повертаються функцією:
```ts
var num: number = 123;
function identity(num: number): number {
    return num;
}
```

### Primitive Types
Примітивні типи JavaScript добре представлені в системі типів TypeScript. Це означає `string`, `number`, `boolean`, як показано нижче:

```ts
var num: number;
var str: string;
var bool: boolean;

num = 123;
num = 123.456;
num = '123'; // Error

str = '123';
str = 123; // Error

bool = true;
bool = false;
bool = 'false'; // Error
```

### Arrays
TypeScript надає спеціальний синтаксис типів для масивів, щоб полегшити вам коментування та документування коду. Синтаксис в основному полягає в постфіксі `[]` до будь-якої дійсної анотації типу (наприклад, `:boolean[]`). Це дозволяє безпечно виконувати будь-які маніпуляції з масивом, які ви робите зазвичай, і захищає вас від помилок, таких як призначення члену неправильного типу. Це показано нижче:

```ts
var boolArray: boolean[];

boolArray = [true, false];
console.log(boolArray[0]); // true
console.log(boolArray.length); // 2
boolArray[1] = true;
boolArray = [false, false];

boolArray[0] = 'false'; // Error!
boolArray = 'false'; // Error!
boolArray = [true, 'false']; // Error!
```

### Interfaces
Інтерфейси є основним способом у TypeScript створювати кілька анотацій типів в одну анотацію з іменем. Розглянемо такий приклад:

```ts
interface Name {
    first: string;
    second: string;
}

var name: Name;
name = {
    first: 'John',
    second: 'Doe'
};

name = {           // Error : `second` is missing
    first: 'John'
};
name = {           // Error : `second` is the wrong type
    first: 'John',
    second: 1337
};
```

Тут ми скомпонували анотації `first: string` + `second: string` у нову анотацію `Name`, яка забезпечує перевірку типу для окремих членів. Інтерфейси мають велику силу в TypeScript, і ми присвятимо цілий розділ тому, як ви можете використовувати це на свою користь.

### Inline Type Annotation
Замість створення нового `інтерфейсу` ви можете анотувати все, що забажаєте *inline* за допомогою `:{ /*Structure*/ }`. Попередній приклад знову представлений із вбудованим типом:

```ts
var name: {
    first: string;
    second: string;
};
name = {
    first: 'John',
    second: 'Doe'
};

name = {           // Error : `second` is missing
    first: 'John'
};
name = {           // Error : `second` is the wrong type
    first: 'John',
    second: 1337
};
```

Вбудовані типи чудово підходять для швидкого надання одноразової анотації типу для чогось. Це позбавить вас від клопоту придумати (потенційно погану) назву типу. Проте, якщо ви виявите, що вставляєте анотацію того самого типу в рядок кілька разів, доцільно розглянути можливість рефакторингу її в інтерфейс (або `type alias` розглянутий далі в цьому розділі).

## Special Types
Крім примітивних типів, які були розглянуті, є кілька типів, які мають особливе значення в TypeScript. Це `any`, `null`, `undefined`, `void`.

### any
Тип `any` займає особливе місце в системі типів TypeScript. Це дає вам вихід із системи типів, щоб наказати компілятору відключитися. `any` сумісний з *any and all* типами в системі типів. Це означає, що *anything can be assigned to it* і *it can be assigned to anything*. Це показано на прикладі нижче:

```ts
var power: any;

// Takes any and all types
power = '123';
power = 123;

// Is compatible with all types
var num: number;
power = num;
num = power;
```

Якщо ви переносите код JavaScript на TypeScript, на початку ви будете близькими друзями з `any`. Однак не сприймайте цю дружбу надто серйозно, оскільки це означає, що *це залежить від вас, щоб забезпечити безпеку типу*. По суті, ви наказуєте компілятору *не робити значущого статичного аналізу*.

### `null` and `undefined`

Те, як вони обробляються системою типів, залежить від прапора компілятора `strictNullChecks` (ми розглянемо цей прапор пізніше). У `strictNullCheck:false` літерали JavaScript `null` і `undefined` ефективно обробляються системою типів так само, як щось типу `any`. Ці літерали можна віднести до будь-якого іншого типу. Це показано на прикладі нижче:

```ts
var num: number;
var str: string;

// These literals can be assigned to anything
num = null;
str = undefined;
```

### `:void`
Використовуй `:void` як прапор, що функция не повертає значення:

```ts
function log(message): void {
    console.log(message);
}
```

## Generics
Багато алгоритмів і структур даних в інформатиці не залежать від *актуального type* об’єкта. Однак ви все одно хочете застосувати обмеження між різними змінними. Простий приклад — це функція, яка приймає список елементів і повертає перевернутий список елементів. Тут існує обмеження між тим, що передається у функцію, і тим, що функція повертає:

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

Тут ви в основному говорите, що функція `reverse` приймає масив (`items: T[]`) *будь-який* типу `T` (зверніть увагу на параметр типу в `reverse<T>`) і повертає масив типу `T` (примітка `: T[]`). Оскільки функція `reverse` повертає елементи того самого типу, що й приймає, TypeScript знає, що змінна `reversed` також має тип `number[]` і забезпечить вам захист від типу. Подібним чином, якщо ви передаєте масив `string[]` до зворотної функції, повернутий результат також буде масивом `string[]`, і ви отримаєте подібну безпеку типу, як показано нижче:

```ts
var strArr = ['1', '2'];
var reversedStrs = reverse(strArr);

reversedStrs = [1, 2]; // Error!
```

Насправді масиви JavaScript вже мають функцію `.reverse`, і TypeScript справді використовує загальні коди для визначення своєї структури:

```ts
interface Array<T> {
 reverse(): T[];
 // ...
}
```

Це означає, що ви отримуєте безпеку типу під час виклику `.reverse` для будь-якого масиву, як показано нижче:

```ts
var numArr = [1, 2];
var reversedNums = numArr.reverse();

reversedNums = ['1', '2']; // Error!
```

Ми обговоримо більше про інтерфейс `Array<T>` пізніше, коли представимо `lib.d.ts` у розділі **Ambient Declarations**.

## Union Type
Досить часто в JavaScript ви хочете дозволити властивості бути одним із кількох типів, наприклад. *a `string` or a `number`*. Тут стане в нагоді *обʼєднаний type* (позначений `|` в анотації типу, наприклад `string|number`). Загальним варіантом використання є функція, яка може приймати один об’єкт або масив об’єктів, наприклад:

```ts
function formatCommandline(command: string[]|string) {
    var line = '';
    if (typeof command === 'string') {
        line = command.trim();
    } else {
        line = command.join(' ').trim();
    }

    // Do stuff with line: string
}
```

## Intersection Type
`extend` — це дуже поширений шаблон у JavaScript, де ви берете два об’єкти та створюєте новий, який має функції обох цих об’єктів. **Intersection Type** дозволяє безпечно використовувати цей шаблон, як показано нижче:

```ts
function extend<T, U>(first: T, second: U): T & U {
  return { ...first, ...second };
}

const x = extend({ a: "hello" }, { b: 42 });

// x now has both `a` and `b`
const a = x.a;
const b = x.b;
```

## Tuple Type
JavaScript не підтримує першокласний кортеж. Люди зазвичай просто використовують масив як кортеж. Це саме те, що підтримує система типів TypeScript. Кортежі можна анотувати за допомогою `: [typeofmember1, typeofmember2]` тощо. Кортеж може мати будь-яку кількість елементів. Кортежі демонструються в наведеному нижче прикладі:


```ts
var nameNumber: [string, number];

// Okay
nameNumber = ['Jenny', 8675309];

// Error!
nameNumber = ['Jenny', '867-5309'];
```

Поєднайте це з підтримкою деструктуризації в TypeScript, кортежі виглядатимуть як першокласні, незважаючи на те, що вони є масивами:

```ts
var nameNumber: [string, number];
nameNumber = ['Jenny', 8675309];

var [name, num] = nameNumber;
```

## Type Alias
TypeScript забезпечує зручний синтаксис для надання імен для анотацій типів, які ви хотіли б використовувати в кількох місцях. Псевдоніми створюються за допомогою синтаксису `type SomeName = someValidTypeAnnotation`. Нижче наведено приклад:

```ts
type StrOrNum = string|number;

// Usage: just like any other notation
var sample: StrOrNum;
sample = 123;
sample = '123';

// Just checking
sample = true; // Error!
```

На відміну від `interface`, ви можете надати псевдонім типу буквально будь-якій анотації типу (корисно для таких речей, як типи об’єднання та перетину). Ось ще кілька прикладів, щоб ви ознайомилися з синтаксисом:

```ts
type Text = string | { text: string };
type Coordinates = [number, number];
type Callback = (data: string) => void;
```

> ПОРАДА: якщо вам потрібні ієрархії анотацій типу, використовуйте `interface`. Їх можна використовувати з `implements` і `extends`

> ПОРАДА: Використовуйте псевдонім типу для простіших структур об’єктів (наприклад,`Coordinates`), щоб дати їм семантичну назву. Крім того, якщо ви хочете дати семантичні назви типам Union або Intersection, використовуйте псевдонім типу.

## Summary
Тепер, коли ви можете почати анотувати більшу частину свого коду JavaScript, ми можемо перейти до найдрібніших деталей усієї потужності, доступної в системі типів TypeScript.
