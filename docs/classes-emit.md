#### Що відбувається з IIFE
js код, згенерований для класу, міг бути таким:
```ts
function Point(x, y) {
    this.x = x;
    this.y = y;
}
Point.prototype.add = function (point) {
    return new Point(this.x + point.x, this.y + point.y);
};
```

Причина, по якій він загорнутий у функцію, що негайно викликається (Immediately-Invoked Function Expression - IIFE)
```ts
(function () {

    // BODY

    return Point;
})();
```

має відношення до спадщини. Це дозволяє TypeScript працювати з базовим класом як зі змінною `_super`.

```ts
var Point3D = (function (_super) {
    __extends(Point3D, _super);
    function Point3D(x, y, z) {
        _super.call(this, x, y);
        this.z = z;
    }
    Point3D.prototype.add = function (point) {
        var point2D = _super.prototype.add.call(this, point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    };
    return Point3D;
})(Point);
```

Зверніть увагу, що IIFE дозволяє TypeScript легко фіксувати базовий клас `Point` у змінній `_super`, яка постійно використовується в тілі класу.

### `__extends`
Розширення
Ви помітите, що як тільки ви успадкуєте клас, TypeScript також генерує таку функцію:
```ts
var __extends = this.__extends || function (d, b) {
    for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
    function __() { this.constructor = d; }
    __.prototype = b.prototype;
    d.prototype = new __();
};
```
Тут `d` відноситься до похідного класу, а `b` відноситься до базового класу. Ця функція робить дві речі:

1. копіює статичні члени базового класу в дочірній клас, тобто `for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];`
2. встановлює прототип дочірнього класу як посилання на батьківське `proto`, тобто фактично `d.prototype.__proto__ = b.prototype`

Люди рідко мають проблеми з розумінням 1 пункту, але багатьом людям важко зрозуміти пункт 2. Тож пояснення є доречним.


#### `d.prototype.__proto__ = b.prototype`

Після того як я навчав багатьох людей цьому, я вважаю наступне пояснення найпростішим. Спочатку ми пояснимо, як код із __extends еквівалентний простому d.prototype.__proto__ = b.prototype, а потім пояснимо, чому цей рядок сам по собі важливий. Щоб все це зрозуміти, потрібно знати такі речі:

1. `__proto__`
2. `prototype`
3. вплив `new` на `this` всередині викликаної функції
4. вплив `new` на `prototype` та `__proto__`

Усі об’єкти в JavaScript містять `__proto__`. Цей елемент часто недоступний у старих браузерах (іноді в документації ця магічна властивість згадується як `[[prototype]]`). Він має одну мету: якщо властивість не знайдено в об’єкті під час пошуку (наприклад, `obj.property`), тоді вона шукається за `obj.__proto__.property`. Якщо його все ще не знайдено, тоді `obj.__proto__.__proto__.property` доки: його не буде знайдено, або останній `.__proto__` сам по собі буде нульовим. Це пояснює, чому кажуть, що JavaScript підтримує успадкування прототипів із коробки. Це показано в наступному прикладі, який можна запустити в консолі Chrome або Node.js:

```ts
var foo = {}

// setup on foo as well as foo.__proto__
foo.bar = 123;
foo.__proto__.bar = 456;

console.log(foo.bar); // 123
delete foo.bar; // remove from object
console.log(foo.bar); // 456
delete foo.__proto__.bar; // remove from foo.__proto__
console.log(foo.bar); // undefined
```

Круто, що ви зрозуміли `__proto__`. Іншим корисним фактом є те, що всі `function` в JavaScript мають властивість під назвою `prototype` і що вони мають `constructor`-член, що вказує на функцію. Це показано нижче:

```ts
function Foo() { }
console.log(Foo.prototype); // {} i.e. it exists and is not undefined
console.log(Foo.prototype.constructor === Foo); // Has a member called `constructor` pointing back to the function
```

Тепер давайте розглянемо вплив `new` на `this` всередині викликаної функції. По суті, `this` всередині викликаної функції вказуватиме на щойно створений об’єкт, який буде повернено функцією. Легко побачити, якщо ви змінюєте властивість `this` всередині функції:

```ts
function Foo() {
    this.bar = 123;
}

// call with the new operator
var newFoo = new Foo();
console.log(newFoo.bar); // 123
```

Єдине, що вам потрібно знати, це те, що виклик `new` для функції призначає `prototype` функції `__proto__` новоствореного об’єкта, який повертається викликом функції. Ось код, який ви можете запустити, щоб повністю зрозуміти це:

```ts
function Foo() { }

var foo = new Foo();

console.log(foo.__proto__ === Foo.prototype); // True!
```

Тепер подивіться на наступне прямо з `__extends`. Я взяв на себе сміливість пронумерувати ці рядки:

```ts
1  function __() { this.constructor = d; }
2  __.prototype = b.prototype;
3  d.prototype = new __();
```

Читання цієї функції у зворотному порядку `d.prototype = new __()` у рядку 3 фактично означає `d.prototype = {__proto__ : __.prototype}` (через вплив new на `prototype` та `__proto__` ), поєднуючи його з попереднім рядком ( тобто рядок 2 `__.prototype = b.prototype;` ) ви отримаєте `d.prototype = {__proto__ : b.prototype}` .

Але зачекайте, ми хотіли `d.prototype.__proto__` , тобто лише прото змінили та зберегли старий `d.prototype.constructor` . Ось де значення першого рядка (тобто `function __() { this.constructor = d; }` ) з’являється. Тут ми фактично матимемо `d.prototype = {__proto__ : __.prototype, constructor : d}` (оскільки вплив `new` на `this` всередині викликаної функції). Отже, оскільки ми відновлюємо `d.prototype.constructor` , єдине, що ми дійсно змінили, це `__proto__`, отже, `d.prototype.__proto__ = b.prototype`.

#### `d.prototype.__proto__ = b.prototype` значення
Значення `d.prototype.__proto__ = b.prototype` 

Важливість полягає в тому, що він дозволяє вам додавати функції-члени до дочірнього класу та успадковувати інші від базового класу. Це демонструє такий простий приклад:

```ts
function Animal() { }
Animal.prototype.walk = function () { console.log('walk') };

function Bird() { }
Bird.prototype.__proto__ = Animal.prototype;
Bird.prototype.fly = function () { console.log('fly') };

var bird = new Bird();
bird.walk();
bird.fly();
```
Загалом `bird.fly` шукатиметься з `bird.__proto__.fly` (пам’ятайте, що `new` робить `bird.__proto__` вказівним на `Bird.prototype` ), а `bird.walk` (успадкований член) шукатиметься з `bird.__proto__.__proto__.walk` (як `bird.__proto__ == Bird.prototype` і `bird.__proto__.__proto__` == `Animal.prototype`).