## Вирази динамічного імпорту

**Вирази динамічного імпорту** - це нова функція та частина **ECMAScript**, яка дозволяє користувачам асинхронно запитувати модуль в будь-який довільний момент у вашій програмі. Комітет JavaScript **TC39** має свою власну пропозицію, яка знаходиться на стадії 3, і називається [пропозиція імпорту для JavaScript](https://github.com/tc39/proposal-dynamic-import).

Альтернативно, упаковувальник **webpack** має функцію, яка називається [**Розбиття коду**](https://webpack.js.org/guides/code-splitting/), яка дозволяє розбити ваш пакет на частини, які можуть бути завантажені асинхронно в пізніший час. Наприклад, це дозволяє спочатку надавати мінімальний пакет запуску, а потім асинхронно завантажувати додаткові функції.

Природньо думати (якщо ми використовуємо webpack у нашому робочому процесі), що [вирази динамічного імпорту TypeScript 2.4](https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript#dynamic-import-expressions) будуть **автоматично створювати** частини пакету та автоматично розбивати ваш JS-кінцевий пакет на частини. АЛЕ це не так просто, як здається, оскільки це залежить від **конфігурації tsconfig.json**, з якою ми працюємо.

Справа в тому, що розбиття коду webpack підтримує дві схожі техніки для досягнення цієї мети: використання **import()** (перевага, пропозиція ECMAScript) та **require.ensure()** (застаріле, специфічне для webpack). І це означає, що очікуваний вихід TypeScript - **залишити оператор import() таким, яким він є**, а не транспілювати його на щось інше.

Давайте розглянемо приклад, щоб з'ясувати, як налаштувати webpack + TypeScript 2.4.

У наступному коді я хочу **ліниво завантажити бібліотеку _moment_**, але мені також цікаво розбиття коду, що означає, що бібліотека moment буде в окремій частині JS (файл JavaScript), який буде завантажуватися лише при потребі.

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

Ось tsconfig.json:

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


**Важливі нотатки**:

- Використання **"module": "esnext"** TypeScript створює імітацію оператора import() для розбиття коду webpack.
- Для отримання додаткової інформації прочитайте цю статтю: [Вирази динамічного імпорту та інтеграція розбиття коду webpack 2 з TypeScript 2.4](https://blog.josequinto.com/2017/06/29/dynamic-import-expressions-and-webpack-code-splitting-integration-with-typescript-2-4/).


Ви можете побачити повний приклад [тут][dynamicimportcode].

[dynamicimportcode]:https://cdn.rawgit.com/basarat/typescript-book/705e4496/code/dynamic-import-expressions/dynamicImportExpression.js