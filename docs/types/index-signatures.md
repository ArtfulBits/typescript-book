# Індексові підписи

Об'єкт в JavaScript (і, отже, в TypeScript) можна отримати доступ за допомогою **рядка**, щоб зберігати посилання на будь-який інший об'єкт JavaScript.

Ось швидкий приклад:

```ts
let foo: any = {};
foo['Hello'] = 'World';
console.log(foo['Hello']); // World
```

Ми зберігаємо рядок `"World"` під ключем `"Hello"`. Пам'ятайте, ми сказали, що він може зберігати будь-який об'єкт JavaScript, тому давайте збережемо екземпляр класу, щоб продемонструвати концепцію:

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

Також пам'ятайте, що ми сказали, що до нього можна отримати доступ за допомогою **рядка**. Якщо ви передаєте будь-який інший об'єкт до індексного підпису, то JavaScript runtime фактично викликає `.toString` на ньому перед отриманням результату. Це продемонстровано нижче:

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

Зверніть увагу, що `toString` буде викликано в будь-який момент, коли `obj` використовується в позиції індексу.

Масиви трохи відрізняються. Для індексування за допомогою `number` JavaScript VM спробує оптимізувати (залежно від таких речей, як чи є це насправді масивом і чи відповідають структури збережених елементів тощо). Тому `number` слід розглядати як дійсний об'єктний аксесор (відмінний від `string`). Ось простий приклад масиву:

```ts
let foo = ['World'];
console.log(foo[0]); // World
```

Отже, це JavaScript. Тепер давайте подивимося на граціозну обробку цього концепту в TypeScript.

## Індексовий підпис TypeScript

Спочатку, оскільки JavaScript *неявно* викликає `toString` на будь-якому об'єкті індексного підпису, TypeScript поверне вам помилку, щоб запобігти початківцям від самостійного пострілу в ногу (я бачу, як користувачі постійно пострілюють собі в ногу, коли використовують JavaScript на stackoverflow):

```ts
let obj = {
  toString(){
    return 'Hello'
  }
}

let foo: any = {};

// ПОМИЛКА: індексний підпис повинен бути рядком, числом ...
foo[obj] = 'World';

// FIX: TypeScript змушує вас бути явним
foo[obj.toString()] = 'World';
```

Причина, чому користувача змушують бути явним, полягає в тому, що типова реалізація `toString` на об'єкті досить жахлива, наприклад, на v8 вона завжди повертає `[object Object]`:

```ts
let obj = {message:'Hello'}
let foo: any = {};

// ПОМИЛКА: індексний підпис повинен бути рядком, числом ...
foo[obj] = 'World';

// Ось де ви його зберегли!
console.log(foo["[object Object]"]); // World
```

Звичайно, `number` підтримується, оскільки

1. Це необхідно для відмінної підтримки масивів / кортежів.
1. Навіть якщо ви використовуєте його для `obj`, його типова реалізація `toString` є гарною (не `[object Object]`).

Пункт 2 показано нижче:

```ts
console.log((1).toString()); // 1
console.log((2).toString()); // 2
```

Тому урок 1:

> Індексні підписи TypeScript повинні бути або `string`, або `number`

Швидке зауваження: `symbols` також є дійсними та підтримуються TypeScript. Але давайте ще не йдемо туди. Давайте робити крок за кроком.

### Оголошення індексного підпису

Таким чином, ми використовували `any`, щоб сказати TypeScript дозволити нам робити все, що ми хочемо. Насправді можна явно вказати *індексний* підпис. Наприклад, скажімо, ви хочете переконатися, що все, що зберігається в об'єкті за допомогою рядка, відповідає структурі `{message: string}`. Це можна зробити з декларацією `{ [index:string] : {message: string} }`. Це продемонстровано нижче:

```ts
let foo:{ [index:string] : {message: string} } = {};

/**
 * Має зберігати речі, які відповідають структурі
 */
/** Ок */
foo['a'] = { message: 'some message' };
/** Помилка: повинен містити `message` типу string. У вас помилка в `message` */
foo['a'] = { messages: 'some message' };

/**
 * Річ, яку читають, також перевіряється на тип
 */
/** Ок */
foo['a'].message;
/** Помилка: повідомлення не існує. У вас помилка в `message` */
foo['a'].messages;
```

> ПОРАДА: назва індексного підпису, наприклад `index` в `{ [index:string] : {message: string} }`, не має значення для TypeScript і призначена лише для зручності читання. наприклад, якщо це імена користувачів, ви можете зробити `{ [username:string] : {message: string} }`, щоб допомогти наступному розробнику, який перегляне код (що може статися саме з вами).

Звичайно, підтримуються також індекси чисел, наприклад `{ [count: number] : SomeOtherTypeYouWantToStoreEgRebate }`

### Всі члени повинні відповідати індексному підпису `string`

Як тільки у вас є індексний підпис `string`, всі явні члени також повинні відповідати цьому індексному підпису. Це показано нижче:

```ts
/** Ок */
interface Foo {
  [key:string]: number;
  x: number;
  y: number;
}
/** Помилка */
interface Bar {
  [key:string]: number;
  x: number;
  y: string; // ПОМИЛКА: Властивість `y` повинна бути типу number
}
```

Це забезпечує безпеку, щоб будь-який рядковий доступ давав той самий результат:

```ts
interface Foo {
  [key:string]: number;
  x: number;
}
let foo: Foo = {x:1,y:2};

// Прямо
foo

```uk
const example: NestedCSS = {
  color: 'red',
  '.subclass': {
    color: 'blue'
  }
}
```

Не мішайте індексатори рядків з *дійсними* значеннями таким чином. Наприклад, помилка в наповненні залишиться непоміченою:

```ts
const failsSilently: NestedCSS = {
  colour: 'red', // Помилки немає, оскільки `colour` є дійсним рядковим селектором
}
```

Замість цього розділіть вкладення на окрему властивість, наприклад, в імені `nest` (або `children` або `subnodes` тощо):

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
  colour: 'red', // Помилка TS: невідома властивість `colour`
}
```

### Виключення деяких властивостей з індексного підпису

Іноді вам потрібно поєднувати властивості в індексний підпис. Цього не рекомендується робити, і ви *повинні* використовувати зазначений вище шаблон вкладеного індексного підпису.

Однак, якщо ви моделюєте *існуючий JavaScript*, ви можете обійти це за допомогою типу перетину. Наступний приклад показує помилку, з якою ви зіткнетеся без використання перетину:

```ts
type FieldState = {
  value: string
}

type FormState = {
  isValid: boolean  // Помилка: не відповідає індексному підпису
  [fieldName: string]: FieldState
}
```

Ось робочий варіант з використанням типу перетину:

```ts
type FieldState = {
  value: string
}

type FormState =
  { isValid: boolean }
  & { [fieldName: string]: FieldState }
```

Зверніть увагу, що, навіть якщо ви можете оголосити його для моделювання існуючого JavaScript, ви не можете створити такий об'єкт за допомогою TypeScript:

```ts
type FieldState = {
  value: string
}

type FormState =
  { isValid: boolean }
  & { [fieldName: string]: FieldState }


// Використовуйте його для деякого об'єкта JavaScript, який ви отримуєте з деякого джерела
declare const foo:FormState; 

const isValidBool = foo.isValid;
const somethingFieldState = foo['something'];

// Використання його для створення об'єкта TypeScript не працюватиме
const bar: FormState = { // Помилка `isValid` не призначається для `FieldState`
  isValid: false
}
```