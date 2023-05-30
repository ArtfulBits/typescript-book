* [Функції-стрілки](#arrow-functions)
* [Tip: переваги стрілочної фукції](#tip-arrow-function-need)
* [Tip: небезпека стрілочної функції](#tip-arrow-function-danger)
* [Tip: бібліотеки, які використовують 'this'](#tip-arrow-functions-with-libraries-that-use-this)
* [Tip: наслідування та стрілочні функції](#tip-arrow-functions-and-inheritance)
* [Tip: Швидке повернення об'єкта](#tip-quick-object-return)

### Arrow Functions
Cтрілочна функція (Arrow Functions)

Її також любовно називаєть "жирною" стрілкою *fat arrow* (оскільки -> тонка стрілка, => жирна стрілка), а також лямбда-функцією (в інших мовах програмування). Найчастіше функція жирної стрілки має вигляд  `()=>something`. 
    Переваги стрілочної функції:
1. Вам не потрібно писати `function`
1. Має контекст `this`
1. Дозволяє передачу значення аргументів

В функціональних мовах програмування, як JavaScript, ви повинні часто уживати `function`. Стрілочна функція спрощує для разробника створення функції
```ts
var inc = (x)=>x+1;
```
`this` традиційно був проблемною точкою в JavaScript. 
Як одного разу сказав мудрий чоловік: «Я ненавиджу JavaScript, оскільки він дуже легко втрачає контекст `this`» . 
Стрілочна функція виправляє це, отримаючи значення `this` з навколишнього контексту. Розглянемо "чистий" клас JavaScript:

```ts
function Person(age) {
    this.age = age;
    this.growOld = function() {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 1, should have been 2
```
Якщо ви запустите цей код у браузері, `this` у функції вказуватиме на `window` , оскільки саме `window` виконує функцію `growOld` . Виправлення полягає у використанні функції стрілки:

```ts
function Person(age) {
    this.age = age;
    this.growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
Причина, чому це працює, полягає в тому, що посилання на `this`  фіксується функцією стрілки з-за меж тіла функції. Це еквівалентно наступному коду JavaScript (це те, що ви б написали самі, якби у вас не було TypeScript):
```ts
function Person(age) {
    this.age = age;
    var _this = this;  // capture this
    this.growOld = function() {
        _this.age++;   // use the captured this
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
Зауважте, що оскільки ви використовуєте TypeScript, ви можете використовувати ще зрічніший синтаксис та поєднувати стрілки з класами:

```ts
class Person {
    constructor(public age:number) {}
    growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

> [Добре відео про цей патерн 🌹](https://egghead.io/lessons/typescript-make-usages-of-this-safe-in-class-methods)

#### Tip: Arrow Function Need
Переваги стрілочної функції
Окрім стислого синтаксису, зручно використовувати функцію-стрілку для виклику з іншого контексту. Подивимося, як це працює:

```ts
var growOld = person.growOld;
// Then later someone else calls it:
growOld();
```
Можно зробити виклик
```ts
person.growOld();
```
тоді `this` буде мати вірний контекст виклику (у цьому прикладі `person`). 

#### Tip: Arrow Function Danger
Небезпека стрілочної функції

В тому випадку, якщо ви не бажаєте, щоб `this` мав значення контексту виклику, вам не слід використовувати функцію-стрілку. Це стосується зворотних викликів (callbacks), які використовуються такими бібліотеками, як jquery, underscore, mocha та інші. Якщо в документації згадуються функції, які використовують `this`, то вам, ймовірно, слід просто використовувати звичайну функцію замість функції-стрілки. Так само, якщо ви плануєте використовувати `arguments`, не використовуйте функцію-стрілку.


#### Tip: Arrow functions with libraries that use `this`
Бібліотеки, які використовують 'this'
Багато бібліотек роблять це, наприклад, `jQuery`ітератори (iterables) - (один із прикладів https://api.jquery.com/jquery.each/) використовують `this`, щоб передати вам об’єкт, який він зараз ітерує. У цьому випадку, якщо ви хочете отримати доступ до бібліотеки, переданої `this`, а також до навколишнього контексту, просто використовуйте тимчасову змінну, як ви робили це без використання стрілочних функцій.


```ts
let _self = this;
something.each(function() {
    console.log(_self); // the lexically scoped value
    console.log(this); // the library passed value
});
```

#### Tip: Arrow functions and inheritance
Наслідування та стрілочні функції
Функції-стрілки, як властивості класів, добре працюють із наслідуванням:

```ts
class Adder {
    constructor(public a: number) {}
    add = (b: number): number => {
        return this.a + b;
    }
}
class Child extends Adder {
    callAdd(b: number) {
        return this.add(b);
    }
}
// Demo to show it works
const child = new Child(123);
console.log(child.callAdd(123)); // 246
```

Однак вони не працюють із ключовим словом super, коли ви намагаєтеся перевизначити функцію в дочірньому класі. Властивості класу зʼєднені з контекстом `this`. Оскільки існує лише один контекст `this`, такі функції не можуть брати участь у виклику `super` (`super` працює лише для
членів прототипу). Ви можете легко обійти це, створивши копію методу перед перевизначенням у дочірньому класі.
```ts
class Adder {
    constructor(public a: number) {}
    // This function is now safe to pass around
    add = (b: number): number => {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    // Create a copy of parent before creating our own
    private superAdd = this.add;
    // Now create our override
    add = (b: number): number => {
        return this.superAdd(b);
    }
}
```

### Tip: Quick object return
Швидке повернення об'єкта
Іноді вам потрібна функція, яка просто повертає літерал об’єкта. Наприклад, щось подібне
```ts
// WRONG WAY TO DO IT
var foo = () => {
    bar: 123
};
```
середовище виконання JavaScript розуміє як *block*, що містить *мітку JavaScript* (через специфікацію JavaScript).


>  Якщо ви не дуже розумієте, що це таке, не хвилюйтеся, оскільки ви все одно отримаєте приємну помилку компілятора від TypeScript із зазначенням «невикористана мітка». Мітки — це стара (та здебільшого невикористовувана) функція JavaScript, яку можна ігнорувати як сучасний GOTO (досвідчені розробники вважають її поганою 🌹)

Ви можете виправити це, оточивши літерал об’єкта за допомогою () :
```ts
// Correct 🌹
var foo = () => ({
    bar: 123
});
```
