# Using Jest with TypeScript

[Професійний урок про Jest / TypeScript](https://egghead.io/lessons/typescript-getting-started-with-jest-using-typescript)


Жодне тестове рішення не є ідеальним. Тим не менш, jеst є чудовим варіантом модульного тестування, який забезпечує чудову підтримку TypeScript.

> Примітка: ми припускаємо, що ви починаєте з простого налаштування вузла package.json. Також усі файли TypeScript мають бути в папці `src`, що завжди рекомендується (навіть без Jest) для чистого налаштування проекту.

## Step 1: Install

Встановіть наступне за допомогою npm:

```shell
npm i jest @types/jest ts-jest typescript -D
```

Пояснення:

* Встановити фреймворк `jest` (`jest`)
* Встановіть типи для `jest` (`@types/jest`)
* Встановіть препроцесор TypeScript для jest (`ts-jest`), який дозволяє jest транспілювати TypeScript на льоту та має вбудовану підтримку source-map.
* Встановіть компілятор TypeScript ('typescript'), який є передумовою для 'ts-jest'.
* Збережіть усе це у своїх залежностях розробника (тестування майже завжди в dev-dependency npm)

## Step 2: Configure Jest

Додайте наступний файл `jest.config.js` до кореня вашого проекту:

```js
module.exports = {
  "roots": [
    "<rootDir>/src"
  ],
  "testMatch": [
    "**/__tests__/**/*.+(ts|tsx|js)",
    "**/?(*.)+(spec|test).+(ts|tsx|js)"
  ],
  "transform": {
    "^.+\\.(ts|tsx)$": "ts-jest"
  },
}
```

(Якщо ваш файл `package.json` містить `"type": "module"`, через що Node припускає, що модулі мають формат es6, ви можете конвертувати наведене вище у формат es6, замінивши верхній рядок на `export default {` .)

Пояснення:

* Ми завжди рекомендуємо зберігати *всі* файли TypeScript у папці `src` вашого проекту. Ми вказуємо це за допомогою параметра `roots`.
* Конфігурація `testMatch` — це глобальний відповідник шаблонів для виявлення файлів .test / .spec у форматі ts / tsx / js.
* Конфігурація `transform` просто повідомляє `jest` використовувати `ts-jest` для файлів ts / tsx.

## Step 3: Run tests

Запустіть `npx jest` з кореня вашого проекту, і jest виконає будь-які ваші тести.

### Optional: Add script target for npm scripts

додайте `package.json`:

```json
{
  "test": "jest"
}
```

* Це дозволяє запускати тести за допомогою простого `npm t`.
* І навіть у режимі перегляду з `npm t -- --watch`.

### Optional: Run jest in watch mode

* `npx jest --watch`

### Example

* для `foo.ts`:


    ```js
    export const sum
      = (...a: number[]) =>
        a.reduce((acc, val) => acc + val, 0);
    ```

* простий `foo.test.ts`:


    ```js
    import { sum } from '../foo';

    test('basic', () => {
      expect(sum()).toBe(0);
    });

    test('basic again', () => {
      expect(sum(1, 2)).toBe(3);
    });
    ```

Примітки:

* Jest надає функцію глобального тестування.
* Jest поставляється попередньо зібраним із твердженнями у формі глобального `expect`.

### Example async

Jest має вбудовану підтримку async/await.


```js
test('basic',async () => {
  expect(sum()).toBe(0);
});

test('basic again', async () => {
  expect(sum(1, 2)).toBe(3);
}, 1000 /* optional timeout */);
```

### Example enzyme

> [Професійний урок-яйцеголовий про Enzyme / Jest / TypeScript](https://egghead.io/lessons/react-test-react-components-and-dom-using-enzyme)

Enzyme дозволяє тестувати реагують компоненти з підтримкою dom. Існує три етапи налаштування:

1. Встановіть enzyme, типи для enzyme, a better snapshot serializer for enzyme, enzyme-adapter-react для вашої версії react `npm i enzyme @types/enzyme enzyme-to-json enzyme-adapter-react-16 -D`
2. Додайте `"snapshotSerializers"` і `"setupTestFrameworkScriptFile"` до вашого `jest.config.js`:  

    ```js
    module.exports = {
      // OTHER PORTIONS AS MENTIONED BEFORE

      // Setup Enzyme
      "snapshotSerializers": ["enzyme-to-json/serializer"],
      "setupFilesAfterEnv": ["<rootDir>/src/setupEnzyme.ts"],
    }
    ```

3. створіть `src/setupEnzyme.ts` файл.

    ```js
    import { configure } from 'enzyme';
    import EnzymeAdapter from 'enzyme-adapter-react-16';
    configure({ adapter: new EnzymeAdapter() });
    ```

Ось приклад компонента react і тесту:

* `checkboxWithLabel.tsx`:

    ```ts
    import * as React from 'react';

    export class CheckboxWithLabel extends React.Component<{
      labelOn: string,
      labelOff: string
    }, {
        isChecked: boolean
      }> {
      constructor(props) {
        super(props);
        this.state = { isChecked: false };
      }

      onChange = () => {
        this.setState({ isChecked: !this.state.isChecked });
      }

      render() {
        return (
          <label>
            <input
              type="checkbox"
              checked={this.state.isChecked}
              onChange={this.onChange}
            />
            {this.state.isChecked ? this.props.labelOn : this.props.labelOff}
          </label>
        );
      }
    }

    ```

* `checkboxWithLabel.test.tsx`:

    ```ts
    import * as React from 'react';
    import { shallow } from 'enzyme';
    import { CheckboxWithLabel } from './checkboxWithLabel';

    test('CheckboxWithLabel changes the text after click', () => {
      const checkbox = shallow(<CheckboxWithLabel labelOn="On" labelOff="Off" />);

      // Interaction demo
      expect(checkbox.text()).toEqual('Off');
      checkbox.find('input').simulate('change');
      expect(checkbox.text()).toEqual('On');

      // Snapshot demo
      expect(checkbox).toMatchSnapshot();
    });
    ```

## Reasons why we like jest

> [Детальнішу інформацію про ці функції див. на веб-сайті jest](http://facebook.github.io/jest/)

* Вбудована бібліотека тверджень.
* Чудова підтримка TypeScript.
* Дуже надійний спостерігач за тестами.
* Тестування Snapshot.
* Вбудовані звіти про покриття.
* Вбудована підтримка async/await.
