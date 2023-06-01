## Замикання

Найкращою рисою, яку отримав JavaScript, є замикання. Функція в JavaScript має доступ до будь-яких змінних, що визначені у зовнішньому області видимості. Замикання найкраще пояснити на прикладах:

```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;

    function bar() {
        console.log(variableInOuterFunction); // Access a variable from the outer scope
    }

    // Call the local function to demonstrate that it has access to arg
    bar();
}

outerFunction("hello closure"); // logs hello closure!
```
Ви можете помітити, що внутрішня функція має доступ до змінної (variableInOuterFunction) зовнішнього контексту. Змінні в зовнішній функції були захоплені (закриті або прив'язані) внутрішньою функцією. Тому й термін **замикання**. Сама концепція достатньо проста й досить інтуїтивна.

Тепер найцікавіша частина: внутрішня функція може мати доступ до змінних зовнішнього контексту *навіть після повернення зовнішньої функції*. Це сталося через те, що змінні все ще прив'язані внутрішньою функцією і не залежать від зовнішньої функції. Знову ж таки, розглянемо приклад:

```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;
    return function() {
        console.log(variableInOuterFunction);
    }
}

var innerFunction = outerFunction("hello closure!");

// Note the outerFunction has returned
innerFunction(); // logs hello closure!
```

### Причина, чому це чудово
Це дозволяє легко компонувати об'єкти, наприклад, за допомогою шаблону розкривання модуля (revealing module pattern):

```ts
function createCounter() {
    let val = 0;
    return {
        increment() { val++ },
        getVal() { return val }
    }
}

let counter = createCounter();
counter.increment();
console.log(counter.getVal()); // 1
counter.increment();
console.log(counter.getVal()); // 2
```

На високому рівні це також те, що робить можливим існування чогось подібного до Node.js (не хвилюйтеся, якщо це наразі не зрозуміло. Зрозуміння прийде з часом 🌹):

```ts
// Pseudo code to explain the concept
server.on(function handler(req, res) {
    loadData(req.id).then(function(data) {
        // the `res` has been closed over and is available
        res.send(data);
    })
});
```
