* [Свіжість](#свіжість)
* [Дозвіл на додаткові властивості](#дозвіл-на-додаткові-властивості)
* [Використання: React](#використання-react-state)

## Свіжість

TypeScript надає концепцію **Свіжості** (також називається *строга перевірка літералів об'єктів*) для спрощення перевірки типів літералів об'єктів, які інакше були б структурно сумісними за типом.

Структурне типування є *надзвичайно зручним*. Розгляньте наступний фрагмент коду. Це дозволяє вам *дуже зручно* оновлювати ваш JavaScript до TypeScript, зберігаючи при цьому рівень типової безпеки:

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

Однак *структурне* типування має слабкість у тому, що воно дозволяє вам помилково думати, що щось приймає більше даних, ніж насправді. Це демонструється в наступному коді, на якому TypeScript видасть помилку, як показано:

```ts
function logName(something: { name: string }) {
    console.log(something.name);
}

logName({ name: 'matt' }); // okay
logName({ name: 'matt', job: 'being awesome' }); // Error: object literals must only specify known properties. `job` is excessive here.
```

Зверніть увагу, що ця помилка виникає *тільки на літералах об'єктів*. Без цієї помилки можна було б подивитися на виклик `logName({ name: 'matt', job: 'being awesome' })` і подумати, що *logName* зробить щось корисне з `job`, тоді як насправді він повністю ігнорує його.

Іншим важливим використанням є інтерфейси з необов'язковими членами. Без такої перевірки літералів об'єктів помилка друкування відповідала б типу. Це демонструється нижче:

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
logIfHasName({neme: 'I just misspelled name to neme'}); // Error: object literals must only specify known properties. `neme` is excessive here.
```

Причина, чому тільки літерали об'єктів перевіряються таким чином, полягає в тому, що в цьому випадку додаткові властивості *які насправді не використовуються* майже завжди є помилкою або неправильним розумінням API.

### Дозвіл на додаткові властивості

Тип може містити підпис індексу, щоб явно вказати, що додаткові властивості дозволені:

```ts
var x: { foo: number, [x: string]: unknown };
x = { foo: 1, baz: 2 };  // Ok, `baz` matched by index signature
```

### Використання: React State

[Facebook ReactJS](https://facebook.github.io/react/) пропонує гарний варіант використання свіжості об'єкта. Досить часто в компоненті ви викликаєте `setState` зі зазначенням лише декількох властивостей замість передачі всіх властивостей, тобто:

```ts
// Припускаючи
interface State {
    foo: string;
    bar: string;
}

// Ви хочете зробити: 
this.setState({foo: "Hello"}); // Error: missing property bar

// Але оскільки стан містить як `foo`, так і `bar`, TypeScript змусить вас зробити: 
this.setState({foo: "Hello", bar: this.state.bar});
```

Використовуючи ідею свіжості, ви позначаєте всі члени як необов'язкові, і *ви все ще можете виявляти помилки друку*!:

```ts
// Припускаючи
interface State {
    foo?: string;
    bar?: string;
}

// Ви хочете зробити: 
this.setState({foo: "Hello"}); // Yay works fine!

// Через свіжість він захищений від помилок друку!
this.setState({foos: "Hello"}); // Error: Objects may only specify known properties

// І все ще перевіряється тип
this.setState({foo: 123}); // Error: Cannot assign number to a string
```