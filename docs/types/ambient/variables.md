### Змінні
Наприклад, щоб повідомити TypeScript про змінну [`process`](https://nodejs.org/api/process.html), ви *можете* зробити наступне:

```ts
declare var process: any;
```

> Вам не *потрібно* цього робити для `process`, оскільки вже існує [спільнота, яка підтримує `node.d.ts`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/node/index.d.ts).

Це дозволяє використовувати змінну `process` без скарг TypeScript:

```ts
process.exit();
```

Ми рекомендуємо використовувати інтерфейси, де це можливо, наприклад:

```ts
interface Process {
    exit(code?: number): void;
}
declare var process: Process;
```

Це дозволяє іншим людям *розширювати* характер глобальних змінних, при цьому все ще повідомляючи TypeScript про такі зміни. Наприклад, розгляньте наступний випадок, де ми додаємо функцію `exitWithLogging` до процесу для нашої розваги:

```ts
interface Process {
    exitWithLogging(code?: number): void;
}
process.exitWithLogging = function() {
    console.log("exiting");
    process.exit.apply(process, arguments);
};
```

Давайте детальніше розглянемо інтерфейси наступного розділу.