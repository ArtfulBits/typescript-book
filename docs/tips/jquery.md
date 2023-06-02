## Поради щодо JQuery

Примітка: для цих порад потрібно встановити файл `jquery.d.ts`

### Швидке визначення нового плагіна

Просто створіть `jquery-foo.d.ts` з таким вмістом:

```ts
interface JQuery {
  foo: any;
}
```

І тепер ви можете використовувати `$('something').foo({whateverYouWant:'hello jquery plugin'})`