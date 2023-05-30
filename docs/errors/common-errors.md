# Поширені помилки
У цьому розділі ми пояснюємо деякі поширені коди помилок, з якими користувачі стикаються в реальному кодуванні.

## TS2304
Приклади:
> `Cannot find name ga`
> `Cannot find name $`
> `Cannot find module jquery`

Можливо, ви використовуєте сторонню бібліотеку (наприклад, google analytics) і не задекларували її. TypeScript намагається вберегти вас від *орфографічних помилок* та *використання змінних без їх оголошення*, тому вам потрібно явно вказати все, що є *доступним під час виконання*, включаючи зовнішні бібліотеки ([детальніше про способи виправлення][ambient]).

## TS2307
Приклад:
> `Cannot find module 'underscore'`

Ви, ймовірно, використовуєте сторонню бібліотеку (наприклад, underscore) як *модуль* ([докладніше про модулі][modules]) але не маєте файлу оголошення для нього ([детальніше про оголошення на ambient][ambient]).

## TS1148
Приклад:
> Cannot compile modules unless the '--module' flag is provided

Перевірте [розділ про модулі][modules].

## Catch clause variable cannot have a type annotation 
Змінна в операторі catch не може мати анотацію типу
<br>
Приклад:
```js
try { something(); }
catch (e: Error) { // Catch clause variable cannot have a type annotation
}
```
TypeScript захищає вас від некоректного JavaScript-коду зі сторони. Використайте замість цього захист типів (type guard):
```js
try { something(); }
catch (e) {
  if (e instanceof Error){
    // Here you go.
  }
}
```

## Interface `ElementClass` cannot simultaneously extend types `Component` and `Component` 
Інтерфейс ElementClass не може одночасно розширювати типи Component та Component. 
<br>
Це відбувається, коли у контексті компіляції є два файлу `react.d.ts` (`@types/react/index.d.ts`).

**Виправити**:
* Видаліть `node_modules` і будь-який `package-lock` (або yarn lock) і знову виконайте `npm install`.
* Якщо це не працює, знайдіть невірний модуль (всі модулі, що використовуються вашим проектом, повинні мати `react.d.ts` як `peerDependency`, а не як жорстку `dependency`) і зазначте це у своєму проекті.


[ambient]: ../types/ambient/d.ts.md
[modules]: ../project/modules.md
