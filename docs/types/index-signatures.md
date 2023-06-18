# Index Signatures

До `Object` у JavaScript (і, отже, TypeScript) можна отримати доступ за допомогою **string** який містить посилання на будь-який інший JavaScript **object**.

Ось короткий приклад:

```ts
let foo: any = {};
foo['Hello'] = 'World';
console.log(foo['Hello']); // World
```

Ми зберігаємо рядок `"World"` під ключем `"Hello"`. Пам’ятайте, ми сказали, що він може зберігати будь-який **object** JavaScript, тому давайте збережемо екземпляр класу, щоб показати концепцію:

```ts
class Foo {
  constructor(public message: string){};
  log(){
    console.log(this.message)
  }
}

let foo: any = {};
foo['Hello'] = new Foo('World');
foo['Hello'].log(); // World
```

акож пам’ятайте, що ми сказали, що до нього можна отримати доступ за допомогою **string**. Якщо ви передаєте будь-який інший об’єкт до сигнатури індексу, середовище виконання JavaScript фактично викликає для нього `.toString` перед отриманням результату. Це показано нижче:

```ts
let obj = {
  toString(){
    console.log('toString called')
    return 'Hello'
  }
}

let foo: any = {};
foo[obj] = 'World'; // toString called
console.log(foo[obj]); // toString called, World
console.log(foo['Hello']); // World
```

Зауважте, що `toString` буде викликатися кожного разу, коли `obj` використовується в позиції індексу.

Масиви трохи відрізняються. Віртуальні машини JavaScript намагатимуться оптимізувати індексацію за числом (залежно від того, чи це насправді масив, чи збігаються структури збережених елементів тощо). Тому `number` слід розглядати як дійсний засіб доступу до об’єкта сам по собі (на відміну від `string`). Ось простий приклад масиву:

```ts
let foo = ['World'];
console.log(foo[0]); // World
```

Отже, це JavaScript. Тепер давайте подивимося, як TypeScript витончено використовує цю концепцію.

## TypeScript Index Signature

По-перше, оскільки JavaScript *implicitly* викликає `toString` для будь-якого підпису індексу об’єкта, TypeScript видасть вам помилку, щоб запобігти початківцям стріляти собі в ногу (я бачу, як користувачі стріляють собі в ногу, коли весь час використовують JavaScript на stackoverflow ):

```ts
let obj = {
  toString(){
    return 'Hello'
  }
}

let foo: any = {};

// ERROR: the index signature must be string, number ...
foo[obj] = 'World';

// FIX: TypeScript forces you to be explicit
foo[obj.toString()] = 'World';
```

Причина примушування користувача бути явним полягає в тому, що реалізація `toString` за замовчуванням для об’єкта є досить жахливою, напр. на v8 завжди повертає `[object Object]`:

```ts
let obj = {message:'Hello'}
let foo: any = {};

// ERROR: the index signature must be string, number ...
foo[obj] = 'World';

// Here is where you actually stored it!
console.log(foo["[object Object]"]); // World
```

Звичайно, `number` підтримується, оскільки

1. це необхідно для відмінної підтримки масиву / кортежу.
1. навіть якщо ви використовуєте його для `obj`, його типова реалізація `toString` є гарною (а не `[object Object]`).

Пункт 2 показано нижче:

```ts
console.log((1).toString()); // 1
console.log((2).toString()); // 2
```

Отже, урок 1:

> Сигнатури індексу TypeScript мають бути або `string` або `number`

Коротка примітка: `symbols` також дійсні та підтримуються TypeScript. Але не будемо поки що туди. Маленькі кроки.

### Declaring an index signature

Тож ми використовували `any`, щоб сказати TypeScript дозволяти нам робити все, що ми хочемо. Насправді ми можемо вказати підпис *index* явно. наприклад скажімо, ви хочете переконатися, що все, що зберігається в об’єкті за допомогою рядка, відповідає структурі `{message: string}`. Це можна зробити за допомогою оголошення `{ [index:string] : {message: string} }`. Це показано нижче:

```ts
let foo:{ [index:string] : {message: string} } = {};

/**
 * Must store stuff that conforms to the structure
 */
/** Ok */
foo['a'] = { message: 'some message' };
/** Error: must contain a `message` of type string. You have a typo in `message` */
foo['a'] = { messages: 'some message' };

/**
 * Stuff that is read is also type checked
 */
/** Ok */
foo['a'].message;
/** Error: messages does not exist. You have a typo in `message` */
foo['a'].messages;
```

> ПОРАДА: назва підпису індексу, напр. `index` в `{ [index:string] : {message: string} }` не має значення для TypeScript і призначений лише для читабельності. напр. якщо це імена користувачів, ви можете зробити `{ [username:string] : {message: string} }`, щоб допомогти наступному розробнику, який перегляне код (яким випадково можете бути ви).

Звичайно, також підтримуються «числові» індекси, напр. `{ [count: number] : SomeOtherTypeYouWantToStoreEgRebate }`

### All members must conform to the `string` index signature

Як тільки у вас є підпис індексу `string` усі явні члени також повинні відповідати цьому підпису індексу. Це показано нижче:

```ts
/** Okay */
interface Foo {
  [key:string]: number;
  x: number;
  y: number;
}
/** Error */
interface Bar {
  [key:string]: number;
  x: number;
  y: string; // ERROR: Property `y` must be of type number
}
```

Це робиться для забезпечення безпеки, щоб будь-який доступ до рядка давав однаковий результат:

```ts
interface Foo {
  [key:string]: number;
  x: number;
}
let foo: Foo = {x:1,y:2};

// Directly
foo['x']; // number

// Indirectly
let x = 'x'
foo[x]; // number
```

### Using a limited set of string literals

Сигнатура індексу може вимагати, щоб рядки індексу були членами об’єднання літеральних рядків за допомогою *Mapped Types* наприклад:

```ts
type Index = 'a' | 'b' | 'c'
type FromIndex = { [k in Index]?: number }

const good: FromIndex = {b:1, c:2}

// Помилка:
// Типу '{ b: число; в: число; d: число; }" не можна призначити типу "FromIndex".
// Літерал об’єкта може вказувати лише відомі властивості, а «d» не існує в типі «FromIndex».
const bad: FromIndex = {b:1, c:2, d:3};
```

Це часто використовується разом із `keyof typeof` для захоплення типів словника, описаного на наступній сторінці.

Специфікація словника може бути відкладена узагальнено:

```ts
type FromSomeIndex<K extends string> = { [key in K]: number }
```

### Having both `string` and `number` indexers

Це не поширений випадок використання, але компілятор TypeScript все одно підтримує його.

Однак він має обмеження, що індексатор `string` є більш суворим, ніж індексатор `number`. Це навмисно, напр. щоб дозволити вводити такі речі, як:

```ts
interface ArrStr {
  [key: string]: string | number; // Must accommodate all members

  [index: number]: string; // Can be a subset of string indexer

  // Just an example member
  length: number;
}
```

### Design Pattern: Nested index signature

> Розгляд API під час додавання підписів індексу

Досить часто в спільноті JS ви бачите API, які зловживають індексаторами рядків. напр. загальний шаблон серед CSS у бібліотеках JS:

```ts
interface NestedCSS {
  color?: string;
  [selector: string]: string | NestedCSS | undefined;
}

const example: NestedCSS = {
  color: 'red',
  '.subclass': {
    color: 'blue'
  }
}
```

Намагайтеся не змішувати рядкові індексатори з *valid* значеннями таким чином. наприклад помилка в заповненні залишиться невиявленою:
```ts
const failsSilently: NestedCSS = {
  colour: 'red', // No error as `colour` is a valid string selector
}
```

Натомість відокремте вкладення у власну властивість, наприклад. в назві на зразок `nest` (або `children` або `subnodes` тощо):

```ts
interface NestedCSS {
  color?: string;
  nest?: {
    [selector: string]: NestedCSS;
  }
}

const example: NestedCSS = {
  color: 'red',
  nest: {
    '.subclass': {
      color: 'blue'
    }
  }
}

const failsSilently: NestedCSS = {
  colour: 'red', // TS Error: unknown property `colour`
}
```

### Excluding certain properties from the index signature

Іноді потрібно об’єднати властивості в підпис індексу. Це не рекомендовано, і ви *повинні* uвикористовувати шаблон підпису вкладеного індексу, згаданий вище.

Однак, якщо ви моделюєте *existing JavaScript* ви можете обійти його за допомогою типу перехрестя. Нижче показано приклад помилки, з якою ви зіткнетеся без використання перехрестя:

```ts
type FieldState = {
  value: string
}

type FormState = {
  isValid: boolean  // Error: Does not conform to the index signature
  [fieldName: string]: FieldState
}
```

Ось обхідний шлях за допомогою типу перетину:

```ts
type FieldState = {
  value: string
}

type FormState =
  { isValid: boolean }
  & { [fieldName: string]: FieldState }
```

Зауважте, що навіть якщо ви можете оголосити його для моделювання існуючого JavaScript, ви не можете створити такий об’єкт за допомогою TypeScript:  

```ts
type FieldState = {
  value: string
}

type FormState =
  { isValid: boolean }
  & { [fieldName: string]: FieldState }


// Use it for some JavaScript object you are getting from somewhere 
declare const foo:FormState; 

const isValidBool = foo.isValid;
const somethingFieldState = foo['something'];

// Using it to create a TypeScript object will not work
const bar: FormState = { // Error `isValid` not assignable to `FieldState
  isValid: false
}
```
