### Ітератори

Ітератор сам по собі не є функцією TypeScript або ES6, ітератор є патерном поведінки, що є загальним для об'єктно-орієнтованих мов програмування. Це, взагалі, об'єкт, який реалізує наступний інтерфейс:

```ts
interface Iterator<T> {
    next(value?: any): IteratorResult<T>;
    return?(value?: any): IteratorResult<T>;
    throw?(e?: any): IteratorResult<T>;
}
```
([Докладніше про те, що означає `<T>` нотація][generics])  
Цей інтерфейс дозволяє отримувати значення з деякої колекції або послідовності, яка належить об'єкту.

`IteratorResult` - це просто пара `value`+`done`: 
```ts
interface IteratorResult<T> {
    done: boolean;
    value: T;
}
```

Уявіть, що є об'єкт деякої рамки, який містить список компонентів, з яких складається ця рамка. З інтерфейсом ітератора можна отримувати компоненти з цього об'єкту рамки, як показано нижче:

```ts
class Component {
  constructor (public name: string) {}
}

class Frame implements Iterator<Component> {

  private pointer = 0;

  constructor(public name: string, public components: Component[]) {}

  public next(): IteratorResult<Component> {
    if (this.pointer < this.components.length) {
      return {
        done: false,
        value: this.components[this.pointer++]
      }
    } else {
      return {
        done: true,
        value: null
      }
    }
  }

}

let frame = new Frame("Door", [new Component("top"), new Component("bottom"), new Component("left"), new Component("right")]);
let iteratorResult1 = frame.next(); //{ done: false, value: Component { name: 'top' } }
let iteratorResult2 = frame.next(); //{ done: false, value: Component { name: 'bottom' } }
let iteratorResult3 = frame.next(); //{ done: false, value: Component { name: 'left' } }
let iteratorResult4 = frame.next(); //{ done: false, value: Component { name: 'right' } }
let iteratorResult5 = frame.next(); //{ done: true, value: null }

// Можливо отримати значення результату ітератора через властивість значення:
let component = iteratorResult1.value; //Component { name: 'top' }
```
Знову ж таки, ітератор сам по собі не є функцією TypeScript, цей код може працювати без явної реалізації інтерфейсів Iterator та IteratorResult. Однак дуже корисно використовувати ці загальні інтерфейси ES6 для забезпечення консистентності коду.

Добре, дуже гарно, але можна зробити ще краще. ES6 визначає *протокол ітератора*, який включає [Symbol.iterator] `symbol`, якщо реалізований інтерфейс Iterable:
```ts
//...
class Frame implements Iterable<Component> {

  constructor(public name: string, public components: Component[]) {}

  [Symbol.iterator]() {
    let pointer = 0;
    let components = this.components;

    return {
      next(): IteratorResult<Component> {
        if (pointer < components.length) {
          return {
            done: false,
            value: components[pointer++]
          }
        } else {
          return {
            done: true,
            value: null
          }
        }
      }
    }
  }
}

let frame = new Frame("Door", [new Component("top"), new Component("bottom"), new Component("left"), new Component("right")]);
for (let cmp of frame) {
  console.log(cmp);
}
```

На жаль, `frame.next()` не буде працювати з цим патерном, і він також виглядає трохи незручно. Для цього є інтерфейс IterableIterator!
```ts
//...
class Frame implements IterableIterator<Component> {

  private pointer = 0;

  constructor(public name: string, public components: Component[]) {}

  public next(): IteratorResult<Component> {
    if (this.pointer < this.components.length) {
      return {
        done: false,
        value: this.components[this.pointer++]
      }
    } else {
      return {
        done: true,
        value: null
      }
    }
  }

  [Symbol.iterator](): IterableIterator<Component> {
    return this;
  }

}
//...
```
Як і з `frame.next()`, так і з циклом `for` тепер можна працювати з інтерфейсом IterableIterator.

Ітератор не обов'язково повинен ітерувати скінченне значення. Типовим прикладом є послідовність Фібоначчі:
```ts
class Fib implements IterableIterator<number> {

  protected fn1 = 0;
  protected fn2 = 1;

  constructor(protected maxValue?: number) {}

  public next(): IteratorResult<number> {
    var current = this.fn1;
    this.fn1 = this.fn2;
    this.fn2 = current + this.fn1;
    if (this.maxValue != null && current >= this.maxValue) {
      return {
        done: true,
        value: null
      } 
    } 
    return {
      done: false,
      value: current
    }
  }

  [Symbol.iterator](): IterableIterator<number> {
    return this;
  }

}

let fib = new Fib();

fib.next() //{ done: false, value: 0 }
fib.next() //{ done: false, value: 1 }
fib.next() //{ done: false, value: 1 }
fib.next() //{ done: false, value: 2 }
fib.next() //{ done: false, value: 3 }
fib.next() //{ done: false, value: 5 }

let fibMax50 = new Fib(50);
console.log(Array.from(fibMax50)); // [ 0, 1, 1, 2, 3, 5, 8, 13, 21, 34 ]

let fibMax21 = new Fib(21);
for(let num of fibMax21) {
  console.log(num); //Друкує послідовність Фібоначчі від 0 до 21
}
```

#### Побудова коду з ітераторами для цільової платформи ES5
Приклади коду вище вимагають ціль ES6. Однак вони можуть працювати
з ціллю ES5, якщо цільовий JS-двигун підтримує `Symbol.iterator`.
Це можна досягти, використовуючи ES6 lib з ціллю ES5
(додайте es6.d.ts до свого проекту), щоб зробити компіляцію.
Скомпільований код повинен працювати в node 4+, Google Chrome та в деяких інших браузерах.

[generics]: ./types/generics.md