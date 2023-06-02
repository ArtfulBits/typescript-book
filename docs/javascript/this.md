## this

Будь-який доступ до ключового слова `this` у функції контролюється способом виклику функції. Його зазвичай називають "контекстом виклику".

Приклад:

```ts
function foo() {
  console.log(this);
}

foo(); // logs out the global e.g. `window` in browsers
let bar = {
  foo
}
bar.foo(); // Logs out `bar` as `foo` was called on `bar`
```

Тому будьте обережні з використанням `this`. Якщо ви хочете від'єднати `this` у класі від контексту виклику, використовуйте функцію-стрілку, [докладніше про це пізніше][arrow].

[arrow]:../arrow-functions.md
