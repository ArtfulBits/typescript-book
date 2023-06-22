## Dynamic import expressions 

**Вирази динамічного імпорту** — це нова функція та частина **ECMAScript**, яка дозволяє користувачам асинхронно запитувати модуль у будь-якій довільній точці вашої програми.
**TC39** Комітет JavaScript має власну пропозицію, яка знаходиться на етапі 3, і вона називається [import() пропозиція для JavaScript](https://github.com/tc39/proposal-dynamic-import).

Крім того, **webpack** bundler має функцію під назвою [**Code Splitting**](https://webpack.js.org/guides/code-splitting/), яка дозволяє вам розділити ваш пакет на частини, які можна завантажено асинхронно пізніше. Наприклад, це дозволяє спочатку обслуговувати мінімальний пакет початкового завантаження, а потім асинхронно завантажувати додаткові функції.

Цілком природно думати (якщо ми використовуємо webpack у нашому робочому процесі розробників), що [вирази динамічного імпорту TypeScript 2.4](https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript#dynamic- import-expressions) **автоматично створюватиме** блоки пакетів і автоматично розділятиме ваш остаточний пакет JS. АЛЕ це не так просто, як здається, оскільки залежить від конфігурації **tsconfig.json**, з якою ми працюємо.

Річ у тім, що при розділенні коду webpack підтримує дві подібні техніки: використання **import()** (бажано, пропозиція ECMAScript) і **require.ensure()** (успадковане, специфічне для webpack). Це означає, що очікуваний результат TypeScript **залишить оператор import() таким, яким він є** замість того, щоб транспілювати його на щось інше.

Давайте розглянемо приклад, щоб зрозуміти, як налаштувати webpack + TypeScript 2.4.

У наступному коді я хочу **ліниво завантажити бібліотеку _moment_**, але мене також цікавить розділення коду, що означає наявність бібліотеки моментів в окремому фрагменті JS (файлу JavaScript), який завантажуватиметься лише за потреби .

```ts
import(/* webpackChunkName: "momentjs" */ "moment")
  .then((moment) => {
      // lazyModule has all of the proper types, autocomplete works,
      // type checking works, code references work \o/
      const time = moment().format();
      console.log("TypeScript >= 2.4.0 Dynamic Import Expression:");
      console.log(time);
  })
  .catch((err) => {
      console.log("Failed to load moment", err);
  });
```

Це tsconfig.json:

```json
{
    "compilerOptions": {
        "target": "es5",                          
        "module": "esnext",                     
        "lib": [
            "dom",
            "es5",
            "scripthost",
            "es2015.promise"
        ],                                        
        "jsx": "react",                           
        "declaration": false,                     
        "sourceMap": true,                        
        "outDir": "./dist/js",                    
        "strict": true,                           
        "moduleResolution": "node",               
        "typeRoots": [
            "./node_modules/@types"
        ],                                        
        "types": [
            "node",
            "react",
            "react-dom"
        ]                                       
    }
}
```


**Важливі примітки**:

- За допомогою **"module": "esnext"** TypeScript створює оператор mimic import(), який буде введено для розділення коду Webpack.
– Для отримання додаткової інформації прочитайте цю статтю: [Інтеграція Dynamic Import Expressions та webpack 2 Code Splitting із TypeScript 2.4](https://blog.josequinto.com/2017/06/29/dynamic-import-expressions-and-webpack-code) -splitting-integration-with-typescript-2-4/).


Повний приклад можна переглянути [тут][динамічний імпорткод].

[dynamicimportcode]:https://cdn.rawgit.com/basarat/typescript-book/705e4496/code/dynamic-import-expressions/dynamicImportExpression.js