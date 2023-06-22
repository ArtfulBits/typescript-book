# Why Cypress
Cypress — чудовий інструмент тестування E2E. Ось кілька вагомих причин розглянути це:

* Можлива ізольована установка.
* Поставляється з підтримкою TypeScript із коробки.
* Забезпечує гарний інтерактивний досвід налагодження google chrome. Це дуже схоже на те, як розробники інтерфейсу користувача здебільшого працюють вручну.
* Має поділ команди на виконання, що забезпечує більш потужне налагодження та тестування стабільності (докладніше про це нижче).
* Містить неявні твердження для забезпечення більш значущого досвіду налагодження з менш крихкими тестами (докладніше про це в порадах нижче).
* Надає можливість легко створювати та спостерігати за серверними XHR, не змінюючи код програми (докладніше про це в порадах нижче).

## Installation
Кроки, надані в цьому процесі інсталяції, дадуть вам гарну папку `e2e`, яку ви можете скопіювати/вставити або як шаблон для вашої організації.

> Ті самі дії представлені у відеоформаті на моєму [каналі YouTube](https://www.youtube.com/watch?v=n3SvvZSWwfM).

Створіть каталог e2e, встановіть cypress, TypeScript і налаштуйте файли конфігурації typescript і cypress:

```sh
mkdir e2e
cd e2e
npm init -y
npm install cypress typescript
npx tsc --init --types cypress --lib dom,es6
echo {} > cypress.json 
```

> Ось кілька причин для створення окремої папки `e2e` спеціально для cypress:
* Створення окремого каталогу або `e2e` полегшує ізоляцію його залежностей `package.json` від решти вашого проекту. Це призводить до зменшення конфліктів залежностей.
* Фреймворки тестування мають звичку забруднювати глобальний простір імен такими речами, як `describe` `it` `expect`. Найкраще зберігати `tsconfig.json` і `node_modules` e2e у цій спеціальній папці `e2e`, щоб запобігти глобальним конфліктам визначення типу.

Додайте кілька сценаріїв до файлу `e2e/package.json`:

```json
  "scripts": {
    "cypress:open": "cypress open",
    "cypress:run": "cypress run"
  },
```

Напишіть свій перший тест у `cypress/integration/basic.ts`:

```ts
it('should perform basic google search', () => {
  cy.visit('https://google.com');
  cy.get('[name="q"]')
    .type('subscribe')
    .type('{enter}');
});
```

Тепер запустіть `npm run cypress:open` під час розробки та `npm run cypress:run` на вашому сервері збірки 🌹

## More description of key Files
У папці `e2e` у вас є такі файли:

* `/cypress.json`: Налаштувати cypress. За умовчанням порожній, і це все, що вам потрібно.
* Підпапки `/cypress`:
     * `/integration`: усі ваші тести.
         * Не соромтеся створювати тести у вкладених папках для кращої організації, напр. `/someFeatureFolder/something.spec.ts`.

## First test
* створіть файл `/cypress/integration/first.ts` з таким вмістом:

```ts
describe('google search', () => {
  it('should work', () => {
    cy.visit('http://www.google.com');
    cy.get('#lst-ib').type('Hello world{enter}')
  });
});
```

## Running in development
Відкрийте IDE cypress за допомогою такої команди.

```sh
npm run cypress:open
```

І виберіть тест для запуску.

## Running on a build server

Ви можете запустити тести Cypress у режимі ci за допомогою наступної команди.

```sh
npm run cypress:run
```

## Tip: Sharing code between UI and test
Тести Cypress скомпільовані / упаковані та запускаються в браузері. Тому сміливо імпортуйте будь-який код проекту у свій тест.

Наприклад, ви можете поділитися значеннями ідентифікатора між інтерфейсом користувача та тестами, щоб переконатися, що селектори CSS не зламаються:

```js
import { Ids } from '../../../src/app/constants';

// Later
cy.get(`#${Ids.username}`)
  .type('john')
```

## Tip: Creating Page Objects
Створення об’єктів, які надають зручну ручку для всіх взаємодій, які різні тести повинні робити зі сторінкою, є загальною умовою тестування. Ви можете створювати об’єкти сторінки за допомогою класів TypeScript із геттерами та методами, наприклад.

```js
import { Ids } from '../../../src/app/constants';

class LoginPage {
  visit() {
    cy.visit('/login');
  }

  get username() {
    return cy.get(`#${Ids.username}`);
  }
}
const page = new LoginPage();

// Later
page.visit();

page.username.type('john');

```

## Tip: Explicit assertion
Cypress поставляється з (вбудованими) бібліотеками тверджень chai та chai-query, які допомагають тестувати веб-сторінки. Ви використовуєте їх із командою `.should`, яка передається в ланцюжок як рядок, замінюючи `.to.foo` на `should('foo')`, наприклад. з chai-jquery ви б `expect($(#foo)).to.have.text('something')`, з cypress ви б `cy.get('#foo').should('have.text', 'something')`:

```
cy.get('#foo')
  .should('have.text', 'something')
```
> Ви отримуєте інтелектуальну інформацію для ланцюжків `should`, оскільки Cypress поставляється з правильними визначеннями TypeScript 👍🏻

Повний список ланцюжків доступний тут: https://docs.cypress.io/guides/references/assertions.html

Якщо ви хочете щось складне, ви можете навіть використати `should(callback)` і, наприклад,

```
cy.get('div')
  .should(($div) => {
    expect($div).to.have.length(1);
    expect($div[0].className).to.contain('heading');
  })
// This is just an example. Normally you would `.should('have.class', 'heading')
```

> ПОРАДА: cypress також виконує автоматичні повторні спроби під час зворотного виклику, тому вони так само вільні від розшарування, як і стандартні цепочки рядків.

## Tip: Commands and Chaining
Кожен виклик функції в ланцюжку Cypress є "командою". Команда `should` є твердженням. Традиційно починати окрему *категорію* ланцюжків і дій окремо, наприклад,

```ts
// Don't do this
cy.get(/**something*/)
  .should(/**something*/)
  .click()
  .should(/**something*/)
  .get(/**something else*/)
  .should(/**something*/)

// Prefer separating the two gets
cy.get(/**something*/)
  .should(/**something*/)
  .click()
  .should(/**something*/)

cy.get(/**something else*/)
  .should(/**something*/)
```

Деякі інші бібліотеки *оцінюють і запускають* код одночасно. Ці бібліотеки змушують вас мати єдиний ланцюжок, який може бути кошмаром для налагодження з перемішаними селекторами та твердженнями.

Команди Cypress по суті є *деклараціями* для середовища виконання cypress для виконання команд пізніше. Прості слова: Cypress робить це легше.

## Tip: Using `contains` for easier querying

Нижче показано приклад:

```
cy.get('#foo')
   // Коли #foo знайдено наступне:
   .contains('Надіслати')
   .click()
   // ^ продовжуватиме шукати те, що містить текст `Надіслати`, і не вдасться, якщо мине час.
   // ^ Після того, як його знайдено, клацніть вузол HTML, який містить текст `Надіслати`.
```

## Tip: Smart delays and retries
Cypress автоматично чекатиме (і повторюватиме спроби) для багатьох асинхронних речей, наприклад.
```
// Якщо немає запиту на псевдонім `foo`, cypress автоматично чекатиме 4 секунди
cy.wait('@foo')
// Якщо немає елемента з ідентифікатором #foo, cypress автоматично чекатиме 4 секунди та повторюватиме спроби
cy.get('#foo')
```
Це позбавляє вас від необхідності постійно додавати довільну логіку очікування (і повторних спроб) у потік тестового коду.

## Tip: Implicit assertion
Cypress має концепцію неявного твердження. Вони спрацьовують, якщо майбутня команда має помилку через попередню команду. наприклад наступне буде помилкою в `contains` (після автоматичних повторних спроб, звичайно), оскільки нічого знайденого не можна `клацнути`:

```ts
cy.get('#foo')
  // Once #foo is found the following:
  .contains('Submit')
  .click()
  // ^ Error: #foo does not have anything that `contains` `'Submit'`
```

У традиційних фреймворках ви отримаєте жахливу помилку, наприклад, `click` не існує на `null`. У Cypress ви отримуєте гарну помилку `#foo` не містить `Submit`. Ця помилка є формою неявного твердження.

## Tip: Waiting for an HTTP request
Багато тестів традиційно були крихкими через усі довільні тайм-аути, необхідні для XHR, які створює програма. `cy.server` полегшує це
* створити псевдонім для внутрішніх викликів
* дочекатися їх появи

```ts
cy.server()
  .route('POST', 'https://example.com/api/application/load')
  .as('load') // create an alias

// Start test
cy.visit('/')

// wait for the call
cy.wait('@load')

// Now the data is loaded
```

## Tip: Mocking an HTTP request response
Ви також можете легко імітувати відповідь на запит за допомогою `route`:
```ts
cy.server()
  .route('POST', 'https://example.com/api/application/load', /* Example payload response */{success:true});
```

### Tip: Asserting an Http request response
Ви можете стверджувати запити без насмішок, використовуючи `route` `onRequest` / `onResponse`, наприклад.

```ts
cy.route({
  method: 'POST',
  url: 'https://example.com/api/application/load',
  onRequest: (xhr) => {
    // Example assertion
    expect(xhr.request.body.data).to.deep.equal({success:true});
  }
})
```

## Tip: Mocking time
Ви можете використовувати `wait`, щоб призупинити тест на деякий час, наприклад. щоб перевірити екран автоматичного сповіщення "Ви збираєтеся вийти з системи":

```ts
cy.visit('/');
cy.wait(waitMilliseconds);
cy.get('#logoutNotification').should('be.visible');
```

Однак рекомендується імітувати час за допомогою `cy.clock` і пересилати час за допомогою `cy.tick`, наприклад.

```ts
cy.clock();

cy.visit('/');
cy.tick(waitMilliseconds);
cy.get('#logoutNotification').should('be.visible');
```

## Tip: Unit testing application code
Ви також можете використовувати cypress для модульного тестування коду програми в ізоляції, наприклад.

```js
import { once } from '../../../src/app/utils';

// Later
it('should only call function once', () => {
  let called = 0;
  const callMe = once(()=>called++);
  callMe();
  callMe();
  expect(called).to.equal(1);
});
```

## Tip: Mocking in unit testing
Якщо ви використовуєте модульне тестування модулів у своїй програмі, ви можете надати макети за допомогою `cy.stub`, наприклад. якщо ви хочете переконатися, що `navigate` викликається у функції `foo`:

* `foo.ts`

```ts
import { navigate } from 'takeme';

export function foo() {
  navigate('/foo');
}
```

* Ви можете зробити це, як у `some.spec.ts`:

```ts
/// <reference types="cypress"/>

import { foo } from '../../../src/app/foo';
import * as takeme from 'takeme';

describe('should work', () => {
  it('should stub it', () => {
    cy.stub(takeme, 'navigate');
    foo();
    expect(takeme.navigate).to.have.been.calledWith('/foo');
  });
});
```

## Tip: Command - execution separation
Коли ви викликаєте команду cypress (або твердження), наприклад. `cy.get('#something')`, функція негайно повертає без фактичного виконання дії. Що він робить, це інформує тестувальника cypress, що вам потрібно буде виконати (виконати) дію (у цьому випадку `get`) у якийсь момент.

По суті, ви створюєте список команд, який потім буде виконувати бігун. Ви можете перевірити цю команду - відокремлення виконання за допомогою простого тесту, зауважте, що ви побачите, що оператори `start / between / end` `console.log` виконуються безпосередньо перед тим, як програма запуску почне *виконувати* команди:

```ts
/// <reference types="cypress"/>

describe('Hello world', () => {
  it('demonstrate command - execution separation', () => {
    console.log('start');
    cy.visit('http://www.google.com');
    console.log('between');
    cy.get('.gLFyf').type('Hello world');
    console.log('end');
  });
});
```

Це розділення виконання команд має дві великі переваги:
* Виконувач може виконувати команди в *flake-resistant* спосіб з автоматичними повторними спробами та неявними твердженнями.
* Дозволяє писати асинхронний код у синхронному режимі без постійного *ланцюжка*, що призводить до складності підтримки коду.

## Tip: Breakpoint
Автоматичні знімки + журнал команд, згенерований тестом cypress, чудово підходять для налагодження. При цьому ви можете призупинити виконання тесту, якщо хочете.

Спочатку переконайтеся, що у вас відкриті інструменти розробника Chrome (з любов’ю — інструменти розробника) у програмі тестування («CMD + ALT + i» на mac / `F12` на Windows). Коли інструменти розробника відкриються, ви можете повторно запустити тест, і інструменти розробника залишаться відкритими. Якщо у вас відкриті інструменти розробника, ви можете призупинити виконання тесту двома способами:

* Точки зупинки коду програми: використовуйте оператор `debugger` у коді програми, і програма виконання тестів зупиниться на цьому, як і стандартна веб-розробка.
* Точки зупинки тестового коду: ви можете використати команду `.debug()`, і виконання тесту cypress зупиниться на ній. Крім того, ви можете використати оператор `debugger` у зворотному виклику команди `.then`, щоб викликати паузу. наприклад `.then(() => { debugger })`. Ви навіть можете використовувати його, щоб отримати якийсь елемент `cy.get('#foo').then(($ /* посилання на елемент dom */) => { debugger; })` або мережевий виклик, наприклад. `cy.request('https://someurl').then((res /* відповідь мережі */) => { налагоджувач });`. Однак ідіоматичним способом є `cy.get('#foo').debug()`, а потім, коли програму тестування призупинено на `debug`, ви можете натиснути `get` у журналі команд, щоб автоматично `console.log` будь-яка інформація, яка вам може знадобитися про команду `.get('#foo')` (та аналогічно для будь-яких інших команд, які ви хочете налагодити).

## Tip: Start server and test
Якщо вам потрібно запустити локальний сервер перед виконанням тестів, ви можете додати `start-server-and-test` https://github.com/bahmutov/start-server-and-test як залежність. Це вимагає наступних аргументів
* сценарій npm для *запуску* сервера (він же сервер)
* кінцева точка, щоб перевірити, чи завантажився сервер (він же старт)
* сценарій npm для початку тестування (він же тест)

Приклад package.json:
```json
{
    "scripts": {
        "start-server": "npm start",
        "run-tests": "mocha e2e-spec.js",
        "ci": "start-server-and-test start-server http://localhost:8080 run-tests"
    }
}
```

## Resources
* Website: https://www.cypress.io/
* Write your first cypress test (gives a nice tour of the cypress IDE) : https://docs.cypress.io/guides/getting-started/writing-your-first-test.html
* Setting up a CI environment (e.g. the provided docker image that works out of the box with `cypress run`): https://docs.cypress.io/guides/guides/continuous-integration.html
* Recipes (Lists recipes with descriptions. Click on headings to navigate to the source code for the recipe): https://docs.cypress.io/examples/examples/recipes.html
* Visual Testing : https://docs.cypress.io/guides/tooling/visual-testing.html
* Optionally set a `baseUrl` in cypress.json to [prevent an initial reload that happens after first `visit`.](https://github.com/cypress-io/cypress/issues/2542)
* Code coverage with cypress: [Webcast](https://www.youtube.com/watch?v=C8g5X4vCZJA)
