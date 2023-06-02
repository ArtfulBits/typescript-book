### Контейнер

Вузол AST може бути контейнером. Це визначає типи `SymbolTables`, які матимуть вузол і пов’язаний із ним символ. Контейнер — це абстрактна концепція (тобто не має пов’язаної структури даних). Концепція контейнера зумовлена кількома факторами, одним з яких є перелік(enum) `ContainerFlags`. Функція `getContainerFlags` (у `binder.ts`) керує цим прапором і представлена нижче:

```ts
function getContainerFlags(node: Node): ContainerFlags {
    switch (node.kind) {
        case SyntaxKind.ClassExpression:
        case SyntaxKind.ClassDeclaration:
        case SyntaxKind.InterfaceDeclaration:
        case SyntaxKind.EnumDeclaration:
        case SyntaxKind.TypeLiteral:
        case SyntaxKind.ObjectLiteralExpression:
            return ContainerFlags.IsContainer;

        case SyntaxKind.CallSignature:
        case SyntaxKind.ConstructSignature:
        case SyntaxKind.IndexSignature:
        case SyntaxKind.MethodDeclaration:
        case SyntaxKind.MethodSignature:
        case SyntaxKind.FunctionDeclaration:
        case SyntaxKind.Constructor:
        case SyntaxKind.GetAccessor:
        case SyntaxKind.SetAccessor:
        case SyntaxKind.FunctionType:
        case SyntaxKind.ConstructorType:
        case SyntaxKind.FunctionExpression:
        case SyntaxKind.ArrowFunction:
        case SyntaxKind.ModuleDeclaration:
        case SyntaxKind.SourceFile:
        case SyntaxKind.TypeAliasDeclaration:
            return ContainerFlags.IsContainerWithLocals;

        case SyntaxKind.CatchClause:
        case SyntaxKind.ForStatement:
        case SyntaxKind.ForInStatement:
        case SyntaxKind.ForOfStatement:
        case SyntaxKind.CaseBlock:
            return ContainerFlags.IsBlockScopedContainer;

        case SyntaxKind.Block:
            // do not treat blocks directly inside a function as a block-scoped-container.
            // Locals that reside in this block should go to the function locals. Otherwise 'x'
            // would not appear to be a redeclaration of a block scoped local in the following
            // example:
            //
            //      function foo() {
            //          var x;
            //          let x;
            //      }
            //
            // If we placed 'var x' into the function locals and 'let x' into the locals of
            // the block, then there would be no collision.
            //
            // By not creating a new block-scoped-container here, we ensure that both 'var x'
            // and 'let x' go into the Function-container's locals, and we do get a collision
            // conflict.
            return isFunctionLike(node.parent) ? ContainerFlags.None : ContainerFlags.IsBlockScopedContainer;
    }

    return ContainerFlags.None;
}
```

Вона *лише* викликається з функції `bindChildren` біндера, яка налаштовує вузол як `container` та/або `blockScopedContainer` залежно від результату виконання функції  `getContainerFlags`. Функція `bindChildren` наведена нижче:

```ts
// All container nodes are kept on a linked list in declaration order. This list is used by
// the getLocalNameOfContainer function in the type checker to validate that the local name
// used for a container is unique.
function bindChildren(node: Node) {
    // Before we recurse into a node's children, we first save the existing parent, container
    // and block-container.  Then after we pop out of processing the children, we restore
    // these saved values.
    let saveParent = parent;
    let saveContainer = container;
    let savedBlockScopeContainer = blockScopeContainer;

    // This node will now be set as the parent of all of its children as we recurse into them.
    parent = node;
```

```uk
// Залежно від того, який тип вузла це, ми можемо змінити поточний контейнер
// та блок-контейнер. Якщо поточний вузол є контейнером, то він автоматично
// вважається поточним блок-контейнером. Також для контейнерів, які можуть містити
// локальні змінні, ми превентивно ініціалізуємо поле .locals. Ми робимо це, оскільки
// дуже ймовірно, що .locals буде потрібно для розміщення деяких дочірніх елементів
// (наприклад, параметр або оголошення змінної).
//
// Однак ми не створюємо .locals превентивно для блок-контейнерів, оскільки це зовсім
// нормально і часто зустрічається, що в блок-контейнерах ніколи не буде змінних з областю
// видимості блоку. Ми не хочемо виділяти об'єкт для кожного "блоку", з яким ми зіткнемося,
// коли більшість з них не буде необхідними.
//
// Нарешті, якщо це блок-контейнер, то ми очищуємо будь-який існуючий об'єкт .locals,
// який він може містити всередині себе. Це стається в інкрементальних сценаріях. Оскільки
// ми можемо використовувати вузол з попередньої компіляції, цей вузол може мати створені
// "локальні" для нього. Ми повинні очистити це, щоб не навмисно переносити застарілі дані
// з попередньої компіляції.
let containerFlags = getContainerFlags(node);
if (containerFlags & ContainerFlags.IsContainer) {
    container = blockScopeContainer = node;

    if (containerFlags & ContainerFlags.HasLocals) {
        container.locals = {};
    }

    addToContainerChain(container);
}

else if (containerFlags & ContainerFlags.IsBlockScopedContainer) {
    blockScopeContainer = node;
    blockScopeContainer.locals = undefined;
}

forEachChild(node, bind);

container = saveContainer;
parent = saveParent;
blockScopeContainer = savedBlockScopeContainer;
```