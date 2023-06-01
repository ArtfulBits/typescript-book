## Null та Undefined

> [Безкоштовне відео на youtube по темі](https://www.youtube.com/watch?v=kaUfBNzuUAI)

JavaScript (і, відповідно, TypeScript) має два нижніх типи: `null` та `undefined`. Вони мають різні значення:

* Щось не було ініціалізовано: `undefined`.
* Щось наразі недоступне: `null`.


### Перевірка на наявність обох.

Справа в тому, що вам доведеться мати справу з обома. Цікаво, що в JavaScript з `==`, `null` і `undefined` рівні тільки один одному:

```ts
// Both null and undefined are only `==` to themselves and each other:
console.log(null == null); // true (of course)
console.log(undefined == undefined); // true (of course)
console.log(null == undefined); // true


// You don't have to worry about falsy values making through this check
console.log(0 == undefined); // false
console.log('' == undefined); // false
console.log(false == undefined); // false
```
Рекомендую використовувати `== null` для перевірки як `undefined`, так і `null`. Зазвичай вам не потрібно робити розрізнення між цими двома значеннями.

```ts
function foo(arg: string | null | undefined) {
  if (arg != null) {
    // arg must be a string as `!=` rules out both null and undefined. 
  }
}
```

> Ви також можете використовувати `== undefined`, але `== null` є більш традиційним/коротшим підходом.

Є одне виключення - це кореневі значення `undefined`, про які ми розповімо далі.

### Перевірка на undefined на рівні кореня (root level)  

Пам'ятаєте, як я говорив, що ви повинні використовувати `== null`? Звичайно, пам'ятаєте (адже я це тільки що сказав ^). Але не використовуйте його для рівня кореня. В строгому режимі, якщо ви використовуєте `foo`, а `foo` є `undefined`, ви отримаєте виняток `ReferenceError` **exception**, і весь стек викликів буде розгорнутий.


> Ви повинні використовувати строгий режим... і, насправді, компілятор TS автоматично вставить його за вас, якщо ви використовуєте модулі... більше про це буде розказано пізніше в книзі, тому вам не потрібно буде явно це вказувати :)

Тому, щоб перевірити, чи змінна визначена на рівні *глобального* контексту, ви зазвичай використовуєте `typeof`:

```ts
if (typeof someglobal !== 'undefined') {
  // someglobal is now safe to use
  console.log(someglobal);
}
```

### Обмежте явне використання `undefined`
Оскільки TypeScript дає вам можливість *документувати* ваші структури окремо від значень, замість використання такого коду, як:

```ts
function foo(){
  // if Something
  return {a:1,b:2};
  // else
  return {a:1,b:undefined};
}
```
ви повинні використовувати анотацію типу:
```ts
function foo():{a:number,b?:number}{
  // if Something
  return {a:1,b:2};
  // else
  return {a:1};
}
```

### Функції зворотного виклику в стилі вузла (Node style callbacks)
Зворотні функції у стилі Node (наприклад, `(err, щосьІнше) => { /* щось */ }`) зазвичай викликаються з `err`, встановленим в `null`, якщо помилок немає. Ви, в будь-якому випадку, зазвичай використовуєте перевірку на істинність для цього:

```ts
fs.readFile('someFile', 'utf8', (err,data) => {
  if (err) {
    // do something
  } else {
    // no error
  }
});
```
При створенні власного API, щоб досягти послідовності, *можна* використовувати `null`   
Насправді, для власних API варто розглянути використання промісів, в цьому випадку вам фактично не потрібно турбуватися про відсутні значення помилок (ви обробляєте їх за допомогою `.then` або `.catch`).

### Не використовуйте `undefined` як засіб *валідаціі*.

Нижче наведен приклад цієї помилки:

```ts
function toInt(str: string) {
  return str ? parseInt(str) : undefined;
}
```
можна набагато краще написати ось так:
```ts
function toInt(str: string): { valid: boolean, int?: number } {
  const int = parseInt(str);
  if (isNaN(int)) {
    return { valid: false };
  }
  else {
    return { valid: true, int };
  }
}
```

### JSON та серіалізація

Стандарт JSON підтримує кодування `null`, але не `undefined`. При кодуванні JSON-об'єкта з атрибутом `null`, атрибут буде включено з його нульовим значенням, тоді як атрибут зі значенням `undefined` буде повністю виключено.

```ts
JSON.stringify({willStay: null, willBeGone: undefined}); // {"willStay":null}
```

Як наслідок, бази даних на основі JSON можуть підтримувати значення `null`, але не `undefined` значення. Оскільки атрибути зі значенням `null` кодуються, ви можете передати намір очистити атрибут, встановивши його значення в `null` перед кодуванням і передачею об'єкта до віддаленого сховища.

Встановлення значень атрибутів до невизначених (undefined) може заощадити витрати на зберігання та передачу даних, оскільки імена атрибутів не будуть закодовані. Однак, це може ускладнити семантику очищених значень у порівнянні з відсутніми.

### Заключні думки
Команда розробників TypeScript не використовує `null`: [Рекомендації з кодування TypeScript](https://github.com/Microsoft/TypeScript/wiki/Coding-guidelines#null-and-undefined) , і це не викликало жодних проблем. Дуглас Крокфорд вважає, що [`null`, це погана ідея](https://www.youtube.com/watch?v=PSGEjv3Tqo0&feature=youtu.be&t=9m21s) та слід просто використовувати `undefined`.

Однак, в кодових базах у стилі Node.js використовується стандартний підхід використання `null` для аргументів помилок, оскільки воно позначає "щось зараз недоступне". Особисто я не зважаю на розрізнення між цими двома значеннями, оскільки багато проектів використовують бібліотеки з різними підходами і просто відкидають обидва значення за допомогою `== null`.
