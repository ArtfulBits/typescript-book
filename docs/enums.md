* [Enums](#enums)
* [Number Enums and numbers](#number-enums-and-numbers)
* [Number Enums and strings](#number-enums-and-strings)
* [Changing the number associated with a number enum](#changing-the-number-associated-with-a-number-enum)
* [Enums are open ended](#enums-are-open-ended)
* [Number Enums as flags](#number-enums-as-flags)
* [String Enums](#string-enums)
* [Const enums](#const-enums)
* [Enum with static functions](#enum-with-static-functions)

### Enums
Перелік.

Перелік (Enums) — це спосіб організувати колекцію пов’язаних значень. Багато інших мов програмування (C/C#/Java) мають тип даних `enum` , але JavaScript його не має. Однак TypeScript його має. Ось приклад визначення переліку TypeScript:

```ts
enum CardSuit {
	Clubs,
	Diamonds,
	Hearts,
	Spades
}

// Зразок використання
var card = CardSuit.Clubs;

// Безпека
card = "not a member of card suit"; // Помилка: рядок не можна призначити типу `CardSuit`
```

Ці значення переліків є `number` , тому відтепер я буду називати їх Числовим Переліком

#### Number Enums and Numbers
Числові Перелікі і числа

Переліки TypeScript базуються на числах. Це означає, що числа можна призначити екземпляру enum, а також будь-що інше, сумісне з `number`

```ts
enum Color {
    Red, // під капотом це елемент 0 = Red
    Green,
    Blue
}
var col = Color.Red;
col = 0; // Фактично те саме, що Color.Red
```

#### Number Enums and Strings
Числові Переліки і рядки

Перш ніж ми подивимося далі на enum, давайте подивимося на JavaScript, який він генерує, ось зразок TypeScript:

```ts
enum Tristate {
    False,
    True,
    Unknown
}
```
генерує такий JavaScript:

```js
var Tristate;
(function (Tristate) {
    Tristate[Tristate["False"] = 0] = "False";
    Tristate[Tristate["True"] = 1] = "True";
    Tristate[Tristate["Unknown"] = 2] = "Unknown";
})(Tristate || (Tristate = {}));
```

зосередимося на рядку Tristate`[Tristate["False"] = 0] = "False";` . У ньому `Tristate["False"] = 0` має бути зрозумілим, тобто встановлює значення "False" для змінної Tristate рівним 0 . Зверніть увагу, що в JavaScript оператор присвоєння повертає присвоєне значення (у цьому випадку 0 ). Тому наступним, що виконується середою виконання JavaScript, є `Tristate[0] = "False"` . Це означає, що ви можете використовувати змінну Tristate для перетворення рядкової версії enum у число або числової версії enum у рядок. Це показано нижче:
```ts
enum Tristate {
    False,
    True,
    Unknown
}
console.log(Tristate[0]); // "Помилковий"
console.log(Tristate["False"]); // 0
console.log(Tristate[Tristate.False]); // "Помилковий" оскільки `Tristate.False == 0`
```

#### Changing the number associated with a Number Enum
Зміна числа, пов’язаного з Числовим Переліком

За замовчуванням переліки починаються з `0` , а потім кожне наступне значення автоматично збільшується на 1. Як приклад розглянемо наступне:
```ts
enum Color {
    Red,     // 0
    Green,   // 1
    Blue     // 2
}
```

Однак ви можете змінити номер, пов’язаний з будь-яким членом переліку, спеціально призначивши його. Це показано нижче, де ми починаємо з 3 і починаємо збільшувати звідти:

```ts
enum Color {
    DarkRed = 3,  // 3
    DarkGreen,    // 4
    DarkBlue      // 5
}
```

> ПОРАДА: я досить часто ініціалізую перший перелік за допомогою `= 1` , оскільки це дає мені змогу безпечно перевірити правдивість значення enum.

#### Number Enums as flags
Числові Переліки як прапори

Одним із чудових способів використання переліків є можливість використовувати переліки як прапори (`Flags`). `Flags` дозволяють перевірити, чи виконується певна умова з набору умов. Розглянемо наступний приклад, де ми маємо набір властивостей тварин:
```ts
enum AnimalFlags {
    None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
    EatsFish       = 1 << 2,
    Endangered     = 1 << 3
}
```

Тут ми використовуємо оператор зсуву вліво, щоб перемістити `1` на певний рівень бітів, щоб отримати побітово непересічні числа `0001 , 0010 , 0100 і 1000` (це десяткові числа `1 , 2 , 4 , 8 ,` якщо вам цікаво). Побітові оператори `|` (або) `/` `&` (та) `/` `~` (не) є вашими найкращими друзями під час роботи з прапорцями, які показано нижче:

```ts
enum AnimalFlags {
    None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
}
type Animal = {
    flags: AnimalFlags
}

function printAnimalAbilities(animal: Animal) {
    var animalFlags = animal.flags;
    if (animalFlags & AnimalFlags.HasClaws) {
        console.log('animal has claws');
    }
    if (animalFlags & AnimalFlags.CanFly) {
        console.log('animal can fly');
    }
    if (animalFlags == AnimalFlags.None) {
        console.log('nothing');
    }
}

let animal: Animal = { flags: AnimalFlags.None };
printAnimalAbilities(animal); // nothing
animal.flags |= AnimalFlags.HasClaws;
printAnimalAbilities(animal); // animal has claws
animal.flags &= ~AnimalFlags.HasClaws;
printAnimalAbilities(animal); // nothing
animal.flags |= AnimalFlags.HasClaws | AnimalFlags.CanFly;
printAnimalAbilities(animal); // animal has claws, animal can fly
```

У цьому прикладі:
* ми використали `|=` для додавання прапорів
* комбінація `&=` і `~` для зняття прапора
* `|` поєднувати прапори

> Примітка: ви можете комбінувати прапорці для створення зручних ярликів у визначенні enum, наприклад `EndangeredFlyingClawedFishEating` нижче:

```ts
enum AnimalFlags {
	None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
    EatsFish       = 1 << 2,
    Endangered     = 1 << 3,

	EndangeredFlyingClawedFishEating = HasClaws | CanFly | EatsFish | Endangered,
}
```

#### String Enums
Строкові переліки

Ми розглядали лише переліки, де значеннями членів є `number`. Вам також дозволено мати члени enum із строковим значеннями. Наприклад:

```ts
export enum EvidenceTypeEnum {
  UNKNOWN = '',
  PASSPORT_VISA = 'passport_visa',
  PASSPORT = 'passport',
  SIGHTED_STUDENT_CARD = 'sighted_tertiary_edu_id',
  SIGHTED_KEYPASS_CARD = 'sighted_keypass_card',
  SIGHTED_PROOF_OF_AGE_CARD = 'sighted_proof_of_age_card',
}
```

З ними легше працювати та налагоджувати, оскільки вони надають значущі рядкові значення, які легше перевіряти. 
Ці значення можна використовувати для простого порівняння рядків.
```ts
// Де `someStringFromBackend` буде '' | 'passport_visa' | 'passport' ... тощо const value = someStringFromBackend as EvidenceTypeEnum;

const value = someStringFromBackend as EvidenceTypeEnum; 

// Зразок використання в коді
if (value === EvidenceTypeEnum.PASSPORT){
    console.log('You provided a passport');
    console.log(value); // `паспорт`
}
```

#### Const Enums
Константні Переліки.

Якщо у вас є таке визначення переліку

```ts
enum Tristate {
    False,
    True,
    Unknown
}

var lie = Tristate.False;
```

то рядок `var lie = Tristate.False` скомпільовано до JavaScript `var lie = Tristate.False` (так, вихідні дані такі ж, як і вхідні). Це означає, що під час виконання буде потрібно знайти `Tristate` , а потім `Tristate.False`. Щоб підвищити продуктивність, ви можете позначити enum як `const enum` . Це показано нижче:
```ts
const enum Tristate {
    False,
    True,
    Unknown
}

var lie = Tristate.False;
```

генерує JavaScript:

```js
var lie = 0;
```

тобто компілятор:

1. *Використовує* при будь-якому зверненню до переліку `0` замість `Tristate.False`.
1. Не створює JavaScript для визначення переліку (немає змінної `Tristate` під час виконання), оскільки його використання вбудовано.

##### Const enum preserveConstEnums

Вбудовування має очевидні переваги продуктивності. Той факт, що під час виконання не існує змінної `Tristate` , просто компілятор допомагає вам, не генеруючи JavaScript, який насправді не використовується під час виконання. Однак ви можете захотіти, щоб компілятор усе ще генерував версію визначення enum у JavaScript для таких речей, як пошук  *number to string* or *string to number* , як ми бачили. У цьому випадку ви можете використати прапорець компілятора `--preserveConstEnums` , і він усе одно генеруватиме визначення var Tristate , щоб ви могли використовувати `Tristate["False"]` або `Tristate[0]` вручну під час виконання, якщо хочете. Це жодним чином не впливає на вбудовування .

### Enum with static functions
### Enum зі статичними функціями

Ви можете використовувати оголошення `enum` + `namespace` , щоб додати статичні методи до enum. Нижче наведено приклад, коли ми додаємо статичний член`isBusinessDay` до переліку  `Weekday`:

```ts
enum Weekday {
	Monday,
	Tuesday,
	Wednesday,
	Thursday,
	Friday,
	Saturday,
	Sunday
}
namespace Weekday {
	export function isBusinessDay(day: Weekday) {
		switch (day) {
			case Weekday.Saturday:
			case Weekday.Sunday:
				return false;
			default:
				return true;
		}
	}
}

const mon = Weekday.Monday;
const sun = Weekday.Sunday;
console.log(Weekday.isBusinessDay(mon)); // true
console.log(Weekday.isBusinessDay(sun)); // false
```

#### Enums are open ended
Enum є відкритими для додавання.

> ПРИМІТКА: відкриті перерахування актуальні, лише якщо ви не використовуєте модулі. Але Ви повинні використовувати модулі. Тому цей розділ останній.

Ось згенерований JavaScript для переліку, показаного знову:

```js
var Tristate;
(function (Tristate) {
    Tristate[Tristate["False"] = 0] = "False";
    Tristate[Tristate["True"] = 1] = "True";
    Tristate[Tristate["Unknown"] = 2] = "Unknown";
})(Tristate || (Tristate = {}));
```

Ми вже пояснювали частину  `Tristate[Tristate["False"] = 0] = "False`. Тепер зверніть увагу на навколишній код (`функція (Tristate) { /*код тут*/ })(Tristate || (Tristate = {}))`; зокрема `(Tristate || (Tristate = {}))` частина. По суті, це фіксує локальну змінну TriState , яка або вказуватиме на вже визначене значення Tristate , або ініціалізує його новим порожнім об’єктом {} .
Це означає, що ви можете розділити (і розширити) визначення enum на кілька файлів. Наприклад, нижче ми розділили визначення кольору на два блоки

```ts
enum Color {
    Red,
    Green,
    Blue
}

enum Color {
    DarkRed = 3,
    DarkGreen,
    DarkBlue
}
```

Зауважте, що вам  *обовʼязково* повторно ініціалізувати перший член (тут `DarkRed = 3`) у продовженні переліку, щоб отримати згенерований код, а не стерти значення з попереднього визначення (тобто значення  `0`, `1`, ... тощо). TypeScript попередить вас, якщо ви цього не зробите (повідомлення про помилку `In an enum with multiple declarations, only one declaration can omit an initializer for its first enum element.`).
