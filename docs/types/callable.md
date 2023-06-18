## Callable
Ви можете анотувати виклики як частину типу або інтерфейсу наступним чином

```ts
interface ReturnString {
  (): string
}
```
Примірником такого інтерфейсу буде функція, яка повертає рядок, наприклад.
```ts
declare const foo: ReturnString;
const bar = foo(); //bar виводиться як рядок
```

### Obvious examples
Звичайно, така *callable* анотація також може вказувати будь-які аргументи / необов’язкові аргументи / залишкові аргументи за потреби. Наприклад, ось складний приклад:

```ts
interface Complex {
  (foo: string, bar?: number, ...others: boolean[]): number;
}
```

Інтерфейс може надавати кілька викликаних анотацій для визначення перевантаження функції. Наприклад:

```ts
interface Overloaded {
    (foo: string): string
    (foo: number): number
}

// приклад імплементації
function stringOrNumber(foo: number): number;
function stringOrNumber(foo: string): string;
function stringOrNumber(foo: any): any {
    if (typeof foo === 'number') {
        return foo * foo;
    } else if (typeof foo === 'string') {
        return `hello ${foo}`;
    }
}

const overloaded: Overloaded = stringOrNumber;

// приклад використання
const str = overloaded(''); // тип `str` визначений як `string`
const num = overloaded(123); // тип of `num` визначений як `number`
```

Звичайно, як і тіло *any* інтерфейсу, ви можете використовувати тіло викликаного інтерфейсу як анотацію типу для змінної. Наприклад:

```ts
const overloaded: {
  (foo: string): string
  (foo: number): number
} = (foo: any) => foo;
```

### Arrow Syntax
Стрілочний синтаксис.

Щоб полегшити вказівку сигнатур, які можна викликати, TypeScript також допускає прості анотації типу стрілок. Наприклад, функцію, яка приймає `number` a повертає `string` можна позначити як:

```ts
const simple: (foo: number) => string
    = (foo) => foo.toString();
```

> Єдине обмеження синтаксису стрілок: ви не можете вказати перевантаження. Для перевантажень ви повинні використовувати повний синтаксис `{ (someArgs): someReturn }`.

### Newable

Newable — це лише спеціальний тип анотації типу *callable* із префіксом `new`. Це просто означає, що вам потрібно *викликати* за допомогою `new`, наприклад.

```ts
interface CallMeWithNewToGetString {
  new(): string
}
// використання
declare const Foo: CallMeWithNewToGetString;
const bar = new Foo(); // передбачається, що bar  має тип string
```
