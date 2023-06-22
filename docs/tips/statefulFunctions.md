## Stateful Functions
Загальною особливістю в інших мовах програмування є використання ключового слова `static` для збільшення *lifetim* (а не *scope*) функціональної змінної, щоб вона жила поза викликами функції. Ось зразок `C`, який досягає цього:

```c
void called() {
    static count = 0;
    count++;
    printf("Called : %d", count);
}

int main () {
    called(); // Called : 1
    called(); // Called : 2
    return 0;
}
```

Оскільки JavaScript (або TypeScript) не має статичні функції, ви можете досягти того самого, використовуючи різні абстракції, які обертаються поверх локальної змінної, наприклад. використовуючи `class`:

```ts
const {called} = new class {
    count = 0;
    called = () => {
        this.count++;
        console.log(`Called : ${this.count}`);
    }
};

called(); // Called : 1
called(); // Called : 2
```

> Розробники C++ також намагаються досягти цього за допомогою шаблону, який вони називають `functor`(клас, який замінює оператор `()`).
