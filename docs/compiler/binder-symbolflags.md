### SymbolFlags
Символи мають `SymbolFlags`. Нижче ми приводимо їх, на момент версії TypeScript 2.2.

```ts
    export const enum SymbolFlags {
        None                    = 0,
        FunctionScopedVariable  = 1 << 0,   // Змінна (var) або параметр
        BlockScopedVariable     = 1 << 1,   // Змінна з областю видимості блоку (let або const)
        Property                = 1 << 2,   // Властивість або член переліку
        EnumMember              = 1 << 3,   // Член переліку
        Function                = 1 << 4,   // Функція
        Class                   = 1 << 5,   // Клас
        Interface               = 1 << 6,   // Інтерфейс
        ConstEnum               = 1 << 7,   // Константний перелік
        RegularEnum             = 1 << 8,   // Перелік
        ValueModule             = 1 << 9,   // Інстанційований модуль
        NamespaceModule         = 1 << 10,  // Неінстанційований модуль
        TypeLiteral             = 1 << 11,  // Літерал типу або відображений тип
        ObjectLiteral           = 1 << 12,  // Літерал об'єкта
        Method                  = 1 << 13,  // Метод
        Constructor             = 1 << 14,  // Конструктор
        GetAccessor             = 1 << 15,  // Метод доступу до властивості "get"
        SetAccessor             = 1 << 16,  // Метод доступу до властивості "set"
        Signature               = 1 << 17,  // Сигнатура виклику, конструктора або індексу
        TypeParameter           = 1 << 18,  // Параметр типу
        TypeAlias               = 1 << 19,  // Псевдонім типу
        ExportValue             = 1 << 20,  // Маркер експортованого значення (див. коментар у declareModuleMember в binder)
        ExportType              = 1 << 21,  // Маркер експортованого типу (див. коментар у declareModuleMember в binder)
        ExportNamespace         = 1 << 22,  // Маркер експортованого простору імен (див. коментар у declareModuleMember в binder)
        Alias                   = 1 << 23,  // Псевдонім для іншого символу (див. коментар у isAliasSymbolDeclaration в checker)
        Prototype               = 1 << 24,  // Властивість прототипу (без представлення вихідного коду)
        ExportStar              = 1 << 25,  // Декларація експорту *
        Optional                = 1 << 26,  // Опціональна властивість
        Transient               = 1 << 27,  // Тимчасовий символ (створений під час перевірки типів)

        Enum = RegularEnum | ConstEnum,
        Variable = FunctionScopedVariable | BlockScopedVariable,
        Value = Variable | Property | EnumMember | Function | Class | Enum | ValueModule | Method | GetAccessor | SetAccessor,
        Type = Class | Interface | Enum | EnumMember | TypeLiteral | ObjectLiteral | TypeParameter | TypeAlias,
        Namespace = ValueModule | NamespaceModule | Enum,
        Module = ValueModule | NamespaceModule,
        Accessor = GetAccessor | SetAccessor,

        // Змінні можуть бути перевизначені, але не можуть перевизначати змінну з областю видимості блоку з тим самим ім'ям, або будь-яке інше значення, яке не є змінною, наприклад ValueModule або Class
        FunctionScopedVariableExcludes = Value & ~FunctionScopedVariable,

        // Декларації з областю видимості блоку не дозволяються перевизначати
        // вони не можуть злитися з чимось в просторі значень
        BlockScopedVariableExcludes = Value,

        ParameterExcludes = Value,
        PropertyExcludes = None,
        EnumMemberExcludes = Value | Type,
        FunctionExcludes = Value & ~(Function | ValueModule),
        ClassExcludes = (Value | Type) & ~(ValueModule | Interface), // Об'єднання клас-інтерфейсу виконується в checker.ts
        InterfaceExcludes = Type & ~(Interface | Class),
        RegularEnumExcludes = (Value | Type) & ~(RegularEnum | ValueModule), // Звичайні переліки злиті тільки з звичайними переліками та модулями
        ConstEnumExcludes = (Value | Type) & ~ConstEnum, // Константні переліки злиті тільки з константними переліками
        ValueModuleExcludes = Value & ~(Function | Class | RegularEnum | ValueModule),
        NamespaceModuleExcludes = 0,
        MethodExcludes = Value & ~Method,
        GetAccessorExcludes = Value & ~SetAccessor,
        SetAccessorExcludes = Value & ~GetAccessor,
        TypeParameterExcludes = Type & ~TypeParameter,
        TypeAliasExcludes = Type,
        AliasExcludes = Alias,

        ModuleMember = Variable | Function | Class | Interface | Enum | Module | TypeAlias | Alias,

        ExportHasLocal = Function | Class | Enum | ValueModule,

        HasExports = Class | Enum | Module,
        HasMembers = Class | Interface | TypeLiteral | ObjectLiteral,

        BlockScoped = BlockScopedVariable | Class | Enum,

        PropertyOrAccessor = Property | Accessor,
        Export = ExportNamespace | ExportType | ExportValue,

        ClassMember = Method | Accessor | Property,

```uk
        /* @internal */
        // Набір речей, які ми вважаємо семантично класифікованими. Використовується для прискорення LS під час класифікації.
        Classifiable = Class | Enum | TypeAlias | Interface | TypeParameter | Module,
    }
```

#### ValueModule
`ValueModule // Інстанційований модуль` це SymbolFlag, який використовується для `SourceFile`, якщо він є зовнішнім модулем.