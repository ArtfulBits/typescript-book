### Об'єднання глобального простору імен
У межах функції `initializeTypeChecker` існує наступний код:

```ts
// Ініціалізація глобальної таблиці символів
forEach(host.getSourceFiles(), file => {
    if (!isExternalModule(file)) {
        mergeSymbolTable(globals, file.locals);
    }
});
```

Це в основному об'єднує всі символи `global` в `let globals: SymbolTable = {};` (всередині функції `createTypeChecker`) таблиці символів SymbolTable. Функція `mergeSymbolTable` переважно викликає іншу функцію `mergeSymbol`.