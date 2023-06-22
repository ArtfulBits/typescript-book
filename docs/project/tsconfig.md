### Basic
Почати роботу з tsconfig.json надзвичайно легко, оскільки основний файл, який вам потрібен, це:
```json
{}
```
тобто порожній файл JSON у *root* вашого проекту. Таким чином TypeScript включатиме *all* файли `.ts` у цьому каталозі (і підкаталогах) як частину контексту компіляції. Він також вибере кілька розумних параметрів компілятора за замовчуванням.

### compilerOptions
Ви можете налаштувати параметри компілятора за допомогою `compilerOptions`:

```json
{
  "compilerOptions": {

    /* Basic Options */                       
    "target": "es5",                       /* Версия ECMAScript: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', or 'ESNEXT'. */
    "module": "commonjs",                  /* Модуль генерации кода: 'commonjs', 'amd', 'system', 'umd' or 'es2015'. */
    "lib": [],                             /* Библиотеки в коде:  */
    "allowJs": true,                       /* Allow JavaScript компиляция */
    "checkJs": true,                       /* Перевіряти помилки в .js файлах. */
    "jsx": "preserve",                     /* JSX code генерация: 'preserve', 'react-native', or 'react'. */
    "declaration": true,                   /* генерация '.d.ts'. */
    "sourceMap": true,                     /* генерация '.map'. */
    "outFile": "./",                       /* збирання вихідних параметров в одному файлі. */
    "outDir": "./",                        /* перенаправлення виходного потоку в діректорію. */
    "rootDir": "./",                       /* Діректорія для вхідних параметрів. використовується совмісно з --outDir. */
    "removeComments": true,                /* не сворювати коментарії */
    "noEmit": true,                        /* Do не створювати вихідний поток. */
    "importHelpers": true,                 /* Import emit helpers from 'tslib'. */
    "downlevelIteration": true,            /* підтримка iterables in 'for-of', spread, and destructuring в 'ES5' or 'ES3'. */
    "isolatedModules": true,               /* трансполюівння кожного файлу в окремий модуль(similar to 'ts.transpileModule'). */
                                              
    /* Strict Type-Checking Options */        
    "strict": true,                        /* Enable all strict type-checking options. */
    "noImplicitAny": true,                 /* Створити error для змінної та декларації  'any' типу. */
    "strictNullChecks": true,              /* Enable strict null checks. */
    "noImplicitThis": true,                /* Створити error для 'this' виразу з 'any' типом. */
    "alwaysStrict": true,                  /* Parse in strict mode and emit "use strict" for each source file. */
                                              
    /* Additional Checks */                   
    "noUnusedLocals": true,                /* невикорастані змінні. */
    "noUnusedParameters": true,            /* невикористені параметри. */
    "noImplicitReturns": true,             /* код не повертає значення. */
    "noFallthroughCasesInSwitch": true,    /* Повідомляти про помилки для випадків провалу в операторі switch. */
                                              
    /* Module Resolution Options */           
    "moduleResolution": "node",            /* Вкажіть стратегію модуля: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
    "baseUrl": "./",                       /* Базовий каталог для визначення неабсолютних імен модулів. */
    "paths": {},                           /* Серія записів, які повторно зіставляють імпорт із розташуваннями пошуку відносно 'baseUrl'. */
    "rootDirs": [],                        /* Список кореневих папок, об’єднаний вміст яких представляє структуру проекту під час виконання. */
    "typeRoots": [],                       /* Список папок, з яких потрібно додати визначення типів. */
    "types": [],                           /* Файли декларацій типів, які будуть включені до компіляції. */
    "allowSyntheticDefaultImports": true, /* Дозволити імпорт за замовчуванням із модулів без експорту за замовчуванням. Це не впливає на випуск коду, лише на перевірку типів. */
                                              
    /* Source Map Options */                  
    "sourceRoot": "./", /* Укажіть розташування, де налагоджувач має знаходити файли TypeScript замість вихідних місць. */
     "mapRoot": "./", /* Укажіть розташування, де налагоджувач має знайти файли карт замість згенерованих місць. */
     "inlineSourceMap": true, /* Видавати один файл із вихідними картами замість окремого файлу. */
     "inlineSources": true, /* Видавати джерело разом із вихідними картами в одному файлі; потрібно встановити '--inlineSourceMap' або '--sourceMap'. */
                                              
    /* Experimental Options */                
    "experimentalDecorators": true, /* Вмикає експериментальну підтримку для декораторів ES7. */
     "emitDecoratorMetadata": true /* Вмикає експериментальну підтримку випромінювання метаданих типу для декораторів. */
  }
}
```

Ці (та інші) параметри компілятора будуть обговорені пізніше.
### TypeScript compiler
Хороші IDE мають вбудовану підтримку миттєвої компіляції `ts` до `js`. Однак, якщо ви хочете запустити компілятор TypeScript вручну з командного рядка під час використання `tsconfig.json`, ви можете зробити це кількома способами:
* Просто запустіть `tsc`, і він шукатиме `tsconfig.json` у поточній, а також у всіх батьківських папках, доки не знайде його.
* Виконайте `tsc -p ./path-to-project-directory`. Звичайно, шлях може бути абсолютним або відносним до поточного каталогу.

Ви навіть можете запустити компілятор TypeScript у режимі *спостереження* за допомогою `tsc -w`, і він спостерігатиме за змінами у ваших файлах проекту TypeScript.
