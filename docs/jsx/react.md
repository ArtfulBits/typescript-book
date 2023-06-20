# React JSX

> [Безкоштовна серія відео на youtube про найкращі практики React / TypeScript](https://www.youtube.com/watch?v=7EW67MqgJvs&list=PLYvdvJlnTOjHNayH7MukKbSJ6PueUNkkG)

> [PRO Egghead курс з TypeScript та React](https://egghead.io/courses/use-typescript-to-develop-react-applications)

[![DesignTSX](https://raw.githubusercontent.com/basarat/typescript-book/master/images/designtsx-banner.png)](https://designtsx.com)

## Налаштування

Наш [швидкий старт для браузера допоможе вам в розробці додатків React](../quick/browser.md). Ось основні моменти:

* Використовуйте файли з розширенням `.tsx` (замість `.ts`).
* Використовуйте `"jsx": "react"` у `compilerOptions` вашого `tsconfig.json`.
* Встановіть визначення для JSX та React у вашому проекті: (`npm i -D @types/react @types/react-dom`).
* Імпортуйте react у ваші `.tsx` файли (`import * as React from "react"`).

## Теги HTML vs. Компоненти

React може рендерити або HTML-теги (рядки), або React-компоненти. Виклик JavaScript для цих елементів відрізняється (`React.createElement('div')` проти `React.createElement(MyComponent)`). Це визначається за регістром першої літери. `foo` розглядається як HTML-тег, а `Foo` - як компонент.

## Перевірка типу

### Теги HTML

HTML-тег `foo` повинен мати тип `JSX.IntrinsicElements.foo`. Ці типи вже визначені для всіх основних тегів у файлі `react-jsx.d.ts`, який ви встановили під час інсталяції. Ось приклад вмісту цього файлу:

```ts
declare module JSX {
    interface IntrinsicElements {
        a: React.HTMLAttributes;
        abbr: React.HTMLAttributes;
        div: React.HTMLAttributes;
        span: React.HTMLAttributes;

        /// so on ...
    }
}
```

### Функціональні компоненти

Ви можете визначати функціональні компоненти просто за допомогою інтерфейсу `React.FunctionComponent`. Наприклад:

```ts
type Props = {
  foo: string;
}
const MyComponent: React.FunctionComponent<Props> = (props) => {
    return <span>{props.foo}</span>
}

<MyComponent foo="bar" />
```

### Компоненти Void-функцій (Void Function Components)

Починаючи з [@types/react PR #46643](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/46643),  ви можете використовувати новий тип `React.VoidFunctionComponent` або `React.VFC` якщо ви хочете вказати, що компонент не приймає  `children`. Це проміжне рішення до наступної основної версії описів типів (де VoidFunctionComponent буде застарілим, і за замовчуванням FunctionComponent не буде приймати дітей).

```ts
type Props = { 
  foo: string 
}
// OK now, in future, error
const FunctionComponent: React.FunctionComponent<Props> = ({ foo, children }: Props) => {
    return <div>{foo} {children}</div>; // OK
};
// Error now (children not support), in future, deprecated
const VoidFunctionComponent: React.VoidFunctionComponent<Props> = ({ foo, children }) => {
    return <div>{foo}{children}</div>; 
};
```

### Компоненти класів

Типи компонентів перевіряються на основі властивостей  `props`. Це моделюється після того, як JSX перетворюється, тобто атрибути стають `props` компонента.

Файл `react.d.ts` визначає клас `React.Component<Props, State>`, який ви повинні розширити у свому власному класі і надати свої власні інтерфейси `Props` і `State`. Нижче наведено приклад:

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

### Підказка React JSX: інтерфейс для рендерингу

Реакт може рендерити різні елементи, такі як `JSX` або `string`. Всі ці елементи об'єднані в типі `React.ReactNode`, тому використовуйте його, коли ви хочете приймати рендерні елементи, наприклад:

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

### Підказка React JSX: Прийняти екземпляр компонента

Оголошення типу `React.ReactElement<T>` надається в типових визначеннях React для того, щоб ви могли вказати тип результату інстанціювання класового компонента `<T/>`, наприклад:

```js
class MyAwesomeComponent extends React.Component {
  render() {
    return <div>Hello</div>;
  }
}

const foo: React.ReactElement<MyAwesomeComponent> = <MyAwesomeComponent />; // Okay
const bar: React.ReactElement<MyAwesomeComponent> = <NotMyAwesomeComponent />; // Error!
```

> Звичайно, ви можете використовувати це як анотацію для аргументу функції, а також як `prop member` React компонента.

### Підказка React JSX: Прийміть *компонент*, який може впливати на пропси та рендеритися за допомогою JSX

Тип `React.Component<Props>` об'єднує `React.ComponentClass<P>` та `React.StatelessComponent<P>`, тому ви можете приймати *щось*, що має тип `Props` і рендерити його за допомогою JSX, наприклад:

```ts
const X: React.Component<Props> = foo; // from somewhere

// Render X with some props:
<X {...props}/>;
```

### Підказка React JSX: Загальні компоненти

Це працює саме так, як очікувається. Ось приклад:

```ts
/** A generic component */
type SelectProps<T> = { items: T[] }
class Select<T> extends React.Component<SelectProps<T>, any> { }

/** Usage */
const Form = () => <Select<string> items={['a','b']} />;
```

### Загальні функції

Щось на кшталт наступного працює добре:

```ts
function foo<T>(x: T): T { return x; }
```

Однак, використання узагальненої функції зі стрілкою цього не зробить:

```ts
const foo = <T>(x: T) => x; // ERROR : unclosed `T` tag
```

**Workaround**: Використовуйте `extends` для параметра generic, щоб підказати компілятору, що це узагальнюючий параметр, наприклад:

```ts
const foo = <T extends unknown>(x: T) => x;
```

### Підказка React: Строго типізовані посилання (Strongly Typed Refs)
Ви по суті ініціалізуєте змінну як об'єднання посилання (`ref`) і `null`, а потім ініціалізуєте її як зворотний виклик (callback), наприклад:

```ts
class Example extends React.Component {
  example() {
    // ... something
  }
  
  render() { return <div>Foo</div> }
}


class Use {
  exampleRef: Example | null = null; 
  
  render() {
    return <Example ref={exampleRef => this.exampleRef = exampleRef } />
  }
}
```

І те саме з посиланнями (`ref`) на вбудовані елементи, наприклад: 

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

### Переконання типу (Type Assertions)

В TypeScript існує можливість використовувати переконання типу (type assertion), коли ви явно вказуєте компілятору, що ви знаєте більше про тип, ніж він сам.

Використовуйте синтаксис `as Foo` для переконання типу, як ми рекомендуємо [mentioned before](../types/type-assertion.md#as-foo-vs-foo).

##  Значення за замовчуванням (Default Props)

* Компоненти зі станом та пропсами за замовчуванням: Ви можете вказати TypeScript, що властивість буде надана ззовні (React), використовуючи оператор *null assertion* (це не ідеальний варіант, але це найпростіше мінімальне рішення з мінімальною кількістю *додаткового коду*, яке я зміг придумати).

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

* Для SFC компонентів зі значеннями за замовчуванням рекомендується використовувати прості патерни JavaScript, оскільки вони добре поєднуються з системою типів TypeScript. Ось приклад:

```tsx
const Hello: React.SFC<{
  /**
   * @default 'TypeScript'
   */
  compiler?: string,
  framework: string
}> = ({
  compiler = 'TypeScript', // Default prop
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

Якщо ви використовуєте веб-компонент, визначення типів React за замовчуванням (`@types/react`) не будуть знати про нього. Але ви можете легко оголосити його, наприклад, оголосити веб-компонент з назвою `my-awesome-slider`, який приймає пропси `MyAwesomeSliderProps`: 

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
