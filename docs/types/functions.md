* [Parameter Annotations](#parameter-annotations)
* [Return Type Annotation](#return-type-annotation)
* [Optional Parameters](#optional-parameters)
* [Overloading](#overloading)

## Functions
Функції.

Система типів TypeScript приділяє велику увагу функціям, адже вони є основними будівельними блоками компонованої системи.

### Parameter annotations
Звичайно, ви можете анотувати параметри функції так само, як ви можете анотувати інші змінні:

```ts
// variable annotation
var sampleVariable: { bar: number }

// function parameter annotation
function foo(sampleParameter: { bar: number }) { }
```

Тут я використовував анотації вбудованого типу. Звичайно, ви можете використовувати інтерфейси тощо.

### Return type annotation

Ви можете позначити тип повернення після списку параметрів функції таким самим стилем, як і для змінної, наприклад. `: Foo` у прикладі нижче:

```ts
interface Foo {
    foo: string;
}

// Return type annotated as `: Foo`
function foo(sample: Foo): Foo {
    return sample;
}
```

Звичайно, я використовував тут `interface`, але ви можете використовувати інші анотації, наприклад. вбудовані анотації.

Досить часто вам не *необхідно* коментувати тип повернення функції, оскільки він зазвичай може бути визначений компілятором.

```ts
interface Foo {
    foo: string;
}

function foo(sample: Foo) {
    return sample; // inferred return type 'Foo'
}
```

Однак, як правило, доцільно додати ці анотації, щоб допомогти з помилками, наприклад:

```ts
function foo() {
    return { fou: 'John Doe' }; // Ви можете не знайти цю орфографічну помилку `foo`, доки не стане надто пізно

sendAsJSON(foo());
```

Якщо ви не плануєте нічого повертати з функції, ви можете позначити її як `:void`. Загалом, ви можете відмовитися від `:void` і залишити це механізму висновку.

### Optional Parameters
Ви можете позначити параметр як необов’язковий:

```ts
function foo(bar: number, bas?: string): void {
    // ..
}

foo(123);
foo(123, 'hello');
```

Крім того, ви можете навіть надати значення за замовчуванням (використовуючи `= someValue` після оголошення параметра), яке буде введено для вас, якщо абонент не надає цей аргумент:

```ts
function foo(bar: number, bas: string = 'hello') {
    console.log(bar, bas);
}

foo(123);           // 123, hello
foo(123, 'world');  // 123, world
```

### Overloading
TypeScript дозволяє *декларувати* перевантаження функцій. Це корисно для документації та безпеки типу. Розглянемо наступний код:

```ts
function padding(a: number, b?: number, c?: number, d?: any) {
    if (b === undefined && c === undefined && d === undefined) {
        b = c = d = a;
    }
    else if (c === undefined && d === undefined) {
        c = a;
        d = b;
    }
    return {
        top: a,
        right: b,
        bottom: c,
        left: d
    };
}
```

Якщо ви уважно подивитесь на код, то зрозумієте, що значення «a», «b», «c», «d» змінюється залежно від кількості переданих аргументів. Крім того, функція очікує лише `1`, `2` або `4` аргументи. Ці обмеження можна *зробити обовʼязковими* та *задокументовати* за допомогою перевантаження функцій. Ви просто оголошуєте заголовок функції кілька разів. Останній заголовок функції є фактично активним *внутрішним* тіла функції, але недоступний для зовнішнього світу.

Це показано нижче:

```ts
// Overloads
function padding(all: number);
function padding(topAndBottom: number, leftAndRight: number);
function padding(top: number, right: number, bottom: number, left: number);
// Фактична реалізація, яка є правдивим представленням усіх випадків, які має обробляти тіло функції
function padding(a: number, b?: number, c?: number, d?: number) {
    if (b === undefined && c === undefined && d === undefined) {
        b = c = d = a;
    }
    else if (c === undefined && d === undefined) {
        c = a;
        d = b;
    }
    return {
        top: a,
        right: b,
        bottom: c,
        left: d
    };
}
```

Тут перші три заголовки функцій доступні як дійсні виклики `padding`:

```ts
padding(1); // Okay: all
padding(1,1); // Okay: topAndBottom, leftAndRight
padding(1,1,1,1); // Okay: top, right, bottom, left

padding(1,1,1); // Error: Not a part of the available overloads
```

Звичайно, важливо, щоб кінцева декларація (справжня декларація, яку видно зсередини функції) була сумісною з усіма перевантаженнями. Це пояснюється тим, що це справжня природа викликів функцій, яку має враховувати тіло функції.

> Перевантаження функцій у TypeScript не призводить до накладних витрат під час виконання. Це просто дозволяє вам задокументувати спосіб, яким ви очікуєте виклик функції, а компілятор контролює решту вашого коду.

### Declaring Functions
> Швидка порада: *Type Declarations* описують типи існуючих реалізацій.

Є два способи *декларування* тип функції без надання реалізації. наприклад

```ts
type LongHand = {
    (a: number): number;
};

type ShortHand = (a: number) => number;
```
Наведений вище приклад *практично* еквівалентний. Відмінності існують, коли ви хочете додати перевантаження. Ви можете лише додати перевантаження у версії довгої декларації, наприклад.

```ts
type LongHandAllowsOverloadDeclarations = {
    (a: number): number;
    (a: string): string;
};
```

[](### Type Compatibility)
