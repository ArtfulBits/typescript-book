### Звіт про помилки зв'язування (Binder Error Reporting)

Помилки зв'язування додаються до списку `bindDiagnostics` вихідного файлу.

Прикладом помилки, виявленої під час зв'язування, є використання `eval` або `arguments` як назви змінної в сценарії з включеним режимом `use strict`. Відповідний код наведено повністю нижче (`checkStrictModeEvalOrArguments` викликається з різних місць, стеки викликів походять з `bindWorker` який викликає різні функції для різних `SyntaxKind` вузлів):

```ts
function checkStrictModeEvalOrArguments(contextNode: Node, name: Node) {
    if (name && name.kind === SyntaxKind.Identifier) {
        let identifier = <Identifier>name;
        if (isEvalOrArgumentsIdentifier(identifier)) {
            // Ми перевіряємо, чи назва знаходиться в середині оголошення або виразу класу; якщо так, ми даємо явне повідомлення,
            // в іншому випадку повідомляємо загальне повідомлення про помилку.
            let span = getErrorSpanForNode(file, name);
            file.bindDiagnostics.push(createFileDiagnostic(file, span.start, span.length,
                getStrictModeEvalOrArgumentsMessage(contextNode), identifier.text));
        }
    }
}

function isEvalOrArgumentsIdentifier(node: Node): boolean {
    return node.kind === SyntaxKind.Identifier &&
        ((<Identifier>node).text === "eval" || (<Identifier>node).text === "arguments");
}

function getStrictModeEvalOrArgumentsMessage(node: Node) {
    // Надаємо спеціалізовані повідомлення, щоб допомогти користувачеві зрозуміти, чому ми вважаємо, що вони в режимі строгого режиму.
    if (getContainingClass(node)) {
        return Diagnostics.Invalid_use_of_0_Class_definitions_are_automatically_in_strict_mode;
    }

    if (file.externalModuleIndicator) {
        return Diagnostics.Invalid_use_of_0_Modules_are_automatically_in_strict_mode;
    }

    return Diagnostics.Invalid_use_of_0_in_strict_mode;
}
```