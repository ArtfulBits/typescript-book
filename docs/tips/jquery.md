## JQuery Tips

Увага: вам потрібно встановити `jquery.d.ts` файл для цього

### Quickly define a new plugin 
Швидка реєстрація.

Створіть `jquery-foo.d.ts` з: 

```ts
interface JQuery {
  foo: any;
}
```

тепер ви можете використовувати `$('something').foo({whateverYouWant:'hello jquery plugin'})`
