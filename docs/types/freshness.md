
* [Freshness](#freshness)
* [Allowing extra properties](#allowing-extra-properties)
* [Use Case: React](#use-case-react-state)

## Freshness

TypeScript забезпечує концепцію **Freshness** (також звану *strict object literal checking*) щоб полегшити перевірку літеральних об’єктів, які в іншому випадку були б структурно сумісними з типами.

Структурна типізація *extremely convenient*. Розглянемо наступний фрагмент коду. Це дозволяє *очень удачно* оновити ваш JavaScript до TypeScript, зберігаючи при цьому рівень безпеки типу:

```ts
function logName(something: { name: string }) {
    console.log(something.name);
}

var person = { name: 'matt', job: 'being awesome' };
var animal = { name: 'cow', diet: 'vegan, but has milk of own species' };
var random = { note: `I don't have a name property` };

logName(person); // okay
logName(animal); // okay
logName(random); // Error: property `name` is missing
```

Однак *structural* типізація має слабкість у тому, що вона дозволяє вам оманливо думати, що щось приймає більше даних, ніж насправді. Це продемонстровано в наступному коді, у якому TypeScript генерує помилку, як показано:

```ts
function logName(something: { name: string }) {
    console.log(something.name);
}

logName({ name: 'matt' }); // okay
logName({ name: 'matt', job: 'being awesome' }); // Помилка: літерали об’єктів повинні вказувати лише відомі властивості. "job" тут надмірна.
```

Зауважте, що ця помилка *виникає лише в object literals*. Без цієї помилки можна було б подивитися на виклик `logName({ name: 'matt', job: 'being awesome' })` і подумати, що *logName* зробить щось корисне з `job`, але, як насправді, він повністю ігноруватиме це.

Інший доцільний випадок використання – це інтерфейси, які мають необов’язкові члени, без такої перевірки об’єктного літералу помилка буде цілком нормальною. Це показано нижче:

```ts
function logIfHasName(something: { name?: string }) {
    if (something.name) {
        console.log(something.name);
    }
}
var person = { name: 'matt', job: 'being awesome' };
var animal = { name: 'cow', diet: 'vegan, but has milk of own species' };

logIfHasName(person); // okay
logIfHasName(animal); // okay
logIfHasName({neme: 'I just misspelled name to neme'}); // Error: обʼєкт має лише відомі властивості. `neme` не існує.
```

Причина, чому лише літерали об’єктів перевіряються таким чином, полягає в тому, що в цьому випадку додаткові властивості, *that aren't actually used* майже завжди є помилкою або неправильним розумінням API.

### Allowing extra properties

Тип може містити сигнатуру індексу, щоб явно вказати, що надлишкові властивості дозволені:

```ts
var x: { foo: number, [x: string]: unknown };
x = { foo: 1, baz: 2 };  // Ok, `baz` matched by index signature
```

### Use Case: React State

[Facebook ReactJS](https://facebook.github.io/react/) пропонує гарний варіант використання freshness. Досить часто в компоненті ви викликаєте `setState` лише з кількома властивостями замість того, щоб передати всі властивості, тобто:

```ts
// Assuming
interface State {
    foo: string;
    bar: string;
}

// You want to do: 
this.setState({foo: "Hello"}); // Error: немає властивісті bar

// Але оскільки стан містить і `foo`, і `bar`, TypeScript змусить вас це зробити: 
this.setState({foo: "Hello", bar: this.state.bar});
```

Використовуючи цей принцип, ви б позначили всіх учасників як необов’язкові, і *you still get to catch typos*!:

```ts
// Assuming
interface State {
    foo?: string;
    bar?: string;
}

// You want to do: 
this.setState({foo: "Hello"}); // Yay works fine!

// Завдяки freshness він також захищений від друкарських помилок!
this.setState({foos: "Hello"}); // Error: Об’єкти можуть визначати лише відомі властивості

// перевірка типів
this.setState({foo: 123}); // Error: Неможливо призначити number в string
```
