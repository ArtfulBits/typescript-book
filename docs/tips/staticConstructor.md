# Static Constructors in TypeScript

"Клас" TypeScript (як і "клас" JavaScript) не може мати статичного конструктора. Однак ви можете отримати той самий ефект досить легко, просто викликавши це самостійно:

```ts
class MyClass {
    static initialize() {
        // Initialization
    }
}
MyClass.initialize();
```
