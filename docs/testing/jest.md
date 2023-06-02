# Використання Jest з TypeScript

> [Урок про Jest / TypeScript на egghead](https://egghead.io/lessons/typescript-getting-started-with-jest-using-typescript)

Немає ідеального рішення для тестування. Однак, Jest - відмінний варіант для юніт-тестування, який надає велику підтримку TypeScript.

> Примітка: ми припускаємо, що ви починаєте з простого налаштування `node package.json`. Крім того, всі файли TypeScript повинні бути в папці `src`, що завжди рекомендується (навіть без Jest) для чистого налаштування проекту.

## Крок 1: Встановлення

Встановіть наступне за допомогою npm:

```shell
npm i jest @types/jest ts-jest typescript -D
```

Пояснення:

* Встановіть фреймворк `jest` (`jest`)
* Встановіть типи для `jest` (`@types/jest`)
* Встановіть препроцесор TypeScript для jest (`ts-jest`), який дозволяє jest транспілювати TypeScript на льоту та мати підтримку source-map.
* Встановіть компілятор TypeScript ('typescript'), який є передумовою для 'ts-jest'.
* Збережіть все це у вашій залежності розробника (тестування майже завжди є залежністю розробника npm)

## Крок 2: Налаштування Jest

Додайте наступний файл `jest.config.js` в корінь вашого проекту:

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

(Якщо ваш файл `package.json` містить `"type": "module"`, що змушує Node припускати, що модулі мають формат es6, ви можете перетворити вищезазначене на формат es6, замінивши верхній рядок на `export default {`.)

Пояснення:

* Ми завжди рекомендуємо мати *всі* файли TypeScript в папці `src` у вашому проекті. Ми припускаємо, що це правда і вказуємо це за допомогою параметру `roots`.
* Налаштування `testMatch` - це збірник глобальних шаблонів для виявлення файлів .test / .spec у форматі ts / tsx / js.
* Налаштування `transform` просто повідомляє `jest` використовувати `ts-jest` для файлів ts / tsx.

## Крок 3: Запуск тестів

Запустіть `npx jest` з кореневої папки вашого проекту, і jest виконає будь-які тести, які ви маєте.

### Опціонально: Додайте ціль скрипту для npm scripts

Додайте `package.json`:

```json
{
  "test": "jest"
}
```

* Це дозволяє вам запускати тести за допомогою простої команди `npm t`.
* І навіть у режимі перегляду з `npm t -- --watch`.

### Опціонально: Запустіть jest у режимі перегляду

* `npx jest --watch`

### Приклад

* Для файлу `foo.ts`:

    ```js
    export const sum
      = (...a: number[]) =>
        a.reduce((acc, val) => acc + val, 0);
    ```

* Простий `foo.test.ts`:

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

* Jest надає глобальну функцію `test`.
* Jest поставляється з вбудованими твердженнями у формі глобального `expect`.

### Приклад async

Jest має вбудовану підтримку async/await. наприклад:

```js
test('basic',async () => {
  expect(sum()).toBe(0);
});

test('basic again', async () => {
  expect(sum(1, 2)).toBe(3);
}, 1000 /* optional timeout */);
```

### Приклад enzyme

> [Урок про Enzyme / Jest / TypeScript на egghead](https://egghead.io/lessons/react-test-react-components-and-dom-using-enzyme)

Enzyme дозволяє тестувати компоненти React з підтримкою DOM. Є три кроки для налаштування Enzyme:

1. Встановіть enzyme, типи для enzyme, кращий серіалізатор snapshot для enzyme, enzyme-adapter-react для вашої версії react `npm i enzyme @types/enzyme enzyme-to-json enzyme-adapter-react-16 -D`
2. Додайте `"snapshotSerializers"` та `"setupTestFrameworkScriptFile"` до вашого `jest.config.js`:  

    ```js
    module.exports = {
      // ІНШІ ЧАСТИ, ЯК ВИЩЕЗАЗНАЧЕНО

      // Налаштування Enzyme
      "snapshotSerializers": ["enzyme-to-json/serializer"],
      "setupFilesAfterEnv": ["<rootDir>/src/setupEnzyme.ts"],
    }
    ```

3. Створіть файл `src/setupEnzyme.ts`.

    ```js
    import { configure } from 'enzyme';
    import EnzymeAdapter from 'enzyme-adapter-react-16';
    configure({ adapter: new EnzymeAdapter() });
    ```

Ось приклад компонента React та тесту:

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
    });
    ```

```
## Приклад Snapshot

```javascript
test('приклад тесту', () => {
  const checkbox = createCheckbox();
  expect(checkbox).toMatchSnapshot();
});
```

## Причини, чому нам подобається Jest

> [Детальніше про ці можливості дивіться на сайті Jest](http://facebook.github.io/jest/)

* Вбудована бібліотека стверджень.
* Велика підтримка TypeScript.
* Дуже надійний тестовий спостерігач.
* Тестування за допомогою знімків.
* Вбудовані звіти про покриття коду.
* Вбудована підтримка async/await.