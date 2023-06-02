## Callable
Ви можете анотувати функції як частину типу або інтерфейсу наступним чином:

```ts
interface ReturnString {
  (): string
}
```
Екземпляр такого інтерфейсу буде функцією, яка повертає рядок, наприклад:

```ts
declare const foo: ReturnString;
const bar = foo(); // bar інферується як рядок
```

### Очевидні приклади
Звичайно, така анотація *callable* також може вказувати будь-які аргументи / необов'язкові аргументи / аргументи залишку за потребою. Ось складний приклад:

```ts
interface Complex {
  (foo: string, bar?: number, ...others: boolean[]): number;
}
```

Інтерфейс може надавати кілька анотацій *callable* для вказівки перевантаження функцій. Наприклад:

```ts
interface Overloaded {
    (foo: string): string
    (foo: number): number
}

// Приклад реалізації
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

// Приклад використання
const str = overloaded(''); // тип `str` інферується як `string`
const num = overloaded(123); // тип `num` інферується як `number`
```

Звичайно, як і тіло будь-якого інтерфейсу, ви можете використовувати тіло *callable* інтерфейсу як анотацію типу для змінної. Наприклад:

```ts
const overloaded: {
  (foo: string): string
  (foo: number): number
} = (foo: any) => foo;
```

### Синтаксис стрілки
Щоб зробити анотації підписів викликів простими, TypeScript також дозволяє прості анотації типів стрілки. Наприклад, функція, яка приймає `number` і повертає `string`, може бути анотована як:

```ts
const simple: (foo: number) => string
    = (foo) => foo.toString();
```

> Єдине обмеження синтаксису стрілки: ви не можете вказати перевантаження. Для перевантаження ви повинні використовувати повне тіло синтаксису `{ (someArgs): someReturn }`.

### Newable

Newable - це просто спеціальний тип анотації *callable* з префіксом `new`. Це просто означає, що вам потрібно *викликати* з `new`, наприклад:

```ts
interface CallMeWithNewToGetString {
  new(): string
}
// Використання
declare const Foo: CallMeWithNewToGetString;
const bar = new Foo(); // bar інферується як рядок
```