# React JSX

> [Безкоштовна серія відеоуроків на YouTube про найкращі практики React / TypeScript](https://www.youtube.com/watch?v=7EW67MqgJvs&list=PLYvdvJlnTOjHNayH7MukKbSJ6PueUNkkG)

> [Курс PRO Egghead про TypeScript та React](https://egghead.io/courses/use-typescript-to-develop-react-applications)

[![DesignTSX](https://raw.githubusercontent.com/basarat/typescript-book/master/images/designtsx-banner.png)](https://designtsx.com)

## Налаштування

Наш [швидкий старт для браузера вже налаштовує вас на розробку додатків React](../quick/browser.md). Ось основні моменти.

* Використовуйте файли з розширенням `.tsx` (замість `.ts`).
* Використовуйте `"jsx": "react"` у `compilerOptions` вашого `tsconfig.json`.
* Встановіть визначення для JSX та React у свій проект: (`npm i -D @types/react @types/react-dom`).
* Імпортуйте React у свої файли `.tsx` (`import * as React from "react"`).

## HTML-теги проти компонентів

React може рендерити HTML-теги (рядки) або компоненти React. JavaScript-еміт для цих елементів відрізняється (`React.createElement('div')` проти `React.createElement(MyComponent)`). Це визначається за допомогою *регістру* *першої* літери. `foo` розглядається як HTML-тег, а `Foo` розглядається як компонент.

## Перевірка типів

### HTML-теги

HTML-тег `foo` має бути типу `JSX.IntrinsicElements.foo`. Ці типи вже визначені для всіх основних тегів у файлі `react-jsx.d.ts`, який ви встановили як частину налаштування. Ось зразок вмісту файлу:

```ts
declare module JSX {
    interface IntrinsicElements {
        a: React.HTMLAttributes;
        abbr: React.HTMLAttributes;
        div: React.HTMLAttributes;
        span: React.HTMLAttributes;

        /// так далі ...
    }
}
```

### Функційні компоненти

Ви можете визначити функційні компоненти просто за допомогою інтерфейсу `React.FunctionComponent` наприклад

```ts
type Props = {
  foo: string;
}
const MyComponent: React.FunctionComponent<Props> = (props) => {
    return <span>{props.foo}</span>
}

<MyComponent foo="bar" />
```

### Функційні компоненти без повернення

Зараз, згідно з [@types/react PR #46643](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/46643), ви можете використовувати новий тип `React.VoidFunctionComponent` або `React.VFC`, якщо хочете заявити, що компонент не приймає `children`. Це тимчасове рішення до наступної основної версії визначень типів (де VoidFunctionComponent буде застарілим, а FunctionComponent за замовчуванням не прийматиме дітей).

```ts
type Props = { 
  foo: string 
}
// OK зараз, в майбутньому - помилка
const FunctionComponent: React.FunctionComponent<Props> = ({ foo, children }: Props) => {
    return <div>{foo} {children}</div>; // OK
};
// Помилка зараз (діти не підтримуються), в майбутньому - застаріло
const VoidFunctionComponent: React.VoidFunctionComponent<Props> = ({ foo, children }) => {
    return <div>{foo}{children}</div>; 
};
```

### Класові компоненти

Компоненти перевіряються на відповідність типу на основі властивості `props` компонента. Це моделюється за тим, як JSX перетворюється, тобто атрибути стають `props` компонента.

Файл `react.d.ts` визначає клас `React.Component<Props, State>`, який ви повинні розширити у своєму власному класі, надаючи свої власні інтерфейси `Props` та `State`. Це продемонстровано нижче:

```ts
type Props = {
  foo: string;
}
class MyComponent extends React.Component<Props, {}> {
    render() {
        return <span>{this.props.foo}</span>
    }
}

<MyComponent foo="bar" />
```

### Порада React JSX: Інтерфейс для відображення

React може відображати деякі речі, такі як `JSX` або `string`. Вони всі об'єднуються в тип `React.ReactNode`, тому використовуйте його, коли хочете приймати відображувані елементи, наприклад

```ts
type Props = {
  header: React.ReactNode;
  body: React.ReactNode;
}
class MyComponent extends React.Component<Props, {}> {
    render() {
        return <div>
            {this.props.header}
            {this.props.body}
        </div>;
    }
}

<MyComponent header={<h1>Header</h1>} body={<i>body</i>} />
```

### Порада React JSX: Прийміть екземпляр компонента

Визначення типів React надає `React.ReactElement<T>` для того, щоб дозволити вам анотувати результат інстанціювання класового компонента `<T/>`. наприклад.

```js
class MyAwesomeComponent extends React.Component {
  render() {
    return <div>Hello</div>;
  }
}

const foo: React.ReactElement<MyAwesomeComponent> = <MyAwesomeComponent />; // Okay
const bar: React.ReactElement<MyAwesomeComponent> = <NotMyAwesomeComponent />; // Error!
```

> Звичайно, ви можете використовувати це як анотацію аргумента функції та навіть як член властивості компонента React.

### Порада React JSX: Прийміть *компонент*, який може діяти на властивості та бути відображеним за допомогою JSX

Тип `React.Component<Props>` об'єднує `React.ComponentClass<P> | React.StatelessComponent<P>`, тому ви можете приймати *щось*, що приймає тип `Props` та відображає його за допомогою JSX, наприклад.

```ts
const X: React.Component<Props> = foo; // звідкись

// Відображення X з деякими властивостями:
<X {...props}/>;
```

### Порада React JSX: Загальні компоненти

Це працює точно так, як очікувалося. Ось приклад:

```ts
/** Загальний компонент */
type SelectProps<T> = { items: T[] }
class Select<T> extends React.Component<SelectProps<T>, any> { }

/** Використання */
const Form = () => <Select<string> items={['a','b']} />;
```

### Загальні функції

Щось на зразок наступного працює добре:

```ts
function foo<T>(x: T): T { return x; }
```

Однак використання стрілкової загальної функції не працюватиме:

```ts
const foo = <T>(x: T) => x; // ПОМИЛКА: незакритий тег `T`
```

**Обхідний шлях**: Використовуйте `extends` на загальному параметрі, щоб натякнути компілятору, що це загальний параметр, наприклад:

```ts
const foo = <T extends unknown>(x: T) => x;
```

### Порада React: Сильно типізовані посилання

Ви по суті ініціалізуєте змінну як об'єднання посилання та `null`, а потім ініціалізуєте її як зворотний виклик, наприклад

```ts
class Example extends React.Component {
  example() {
    // ... щось
  }
  
  render() { return <div>Foo</div> }
}

```ts
class Use {
  exampleRef: Example | null = null; 
  
  render() {
    return <Example ref={exampleRef => this.exampleRef = exampleRef } />
  }
}
```

Те ж саме з ref для нативних елементів, наприклад:

```ts
class FocusingInput extends React.Component<{ value: string, onChange: (value: string) => any }, {}>{
  input: HTMLInputElement | null = null;
    
  render() {
    return (
      <input
        ref={(input) => this.input = input}
        value={this.props.value}
        onChange={(e) => { this.props.onChange(e.target.value) } }
        />
      );
    }
    focus() {
      if (this.input != null) { this.input.focus() }
    }
}
```

### Перевірка типів

Використовуйте синтаксис `as Foo` для перевірки типів, як ми [вже згадували](../types/type-assertion.md#as-foo-vs-foo).

## Значення за замовчуванням

* Компоненти зі станом і значеннями за замовчуванням: Ви можете повідомити TypeScript, що властивість буде надана зовнішньо (за допомогою React) за допомогою оператора *null assertion* (це не ідеально, але це найпростіший мінімальний *додатковий код* рішення, яке я міг придумати).

```tsx
class Hello extends React.Component<{
  /**
   * @default 'TypeScript'
   */
  compiler?: string,
  framework: string
}> {
  static defaultProps = {
    compiler: 'TypeScript'
  }
  render() {
    const compiler = this.props.compiler!;
    return (
      <div>
        <div>{compiler}</div>
        <div>{this.props.framework}</div>
      </div>
    );
  }
}

ReactDOM.render(
  <Hello framework="React" />, // TypeScript React
  document.getElementById("root")
);
```

* SFC зі значеннями за замовчуванням: Рекомендуємо використовувати прості шаблони JavaScript, оскільки вони добре працюють з системою типів TypeScript, наприклад:

```tsx
const Hello: React.SFC<{
  /**
   * @default 'TypeScript'
   */
  compiler?: string,
  framework: string
}> = ({
  compiler = 'TypeScript', // Значення за замовчуванням
  framework
}) => {
    return (
      <div>
        <div>{compiler}</div>
        <div>{framework}</div>
      </div>
    );
  };


ReactDOM.render(
  <Hello framework="React" />, // TypeScript React
  document.getElementById("root")
);
```

## Оголошення веб-компонента

Якщо ви використовуєте веб-компонент, типові визначення React за замовчуванням (`@types/react`) не будуть про нього знати. Але ви можете легко оголосити його, наприклад, щоб оголосити веб-компонент під назвою `my-awesome-slider`, який приймає Props `MyAwesomeSliderProps`, ви повинні:

```tsx
declare global {
  namespace JSX {
    interface IntrinsicElements {
      'my-awesome-slider': MyAwesomeSliderProps;
    }

    interface MyAwesomeSliderProps extends React.Attributes {
      name: string;
    }
  }
}
```

Тепер ви можете використовувати його в TSX:

```tsx
<my-awesome-slider name='amazing'/>
```