### Variables
Наприклад, щоб повідомити TypeScript про [`process` variable](https://nodejs.org/api/process.html) you *можете* зробити:

```ts
declare var process: any;
```

> Вам *потрібно* робити це для `process`, оскільки вже існує [community maintained `node.d.ts`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/node/index.d.ts).

Це дозволяє вам використовувати змінну `process` без скарг TypeScript:

```ts
process.exit();
```

Ми рекомендуємо використовувати інтерфейс, де це можливо, наприклад:

```ts
interface Process {
    exit(code?: number): void;
}
declare var process: Process;
```

Це дозволяє іншим людям *розширити* природу цих глобальних змінних, водночас повідомляючи TypeScript про такі зміни. наприклад Розглянемо наступний випадок, коли ми додаємо функцію `exitWithLogging` для обробки для нашої коду:

```ts
interface Process {
    exitWithLogging(code?: number): void;
}
process.exitWithLogging = function() {
    console.log("exiting");
    process.exit.apply(process, arguments);
};
```

Далі розглянемо інтерфейси трохи докладніше.
