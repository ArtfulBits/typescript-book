# Singleton Pattern

Традиційний шаблон singleton насправді використовується для подолання того факту, що весь код має бути в `класі`.

```ts
class Singleton {
    private static instance: Singleton;
    private constructor() {
        // do something construct...
    }
    static getInstance() {
        if (!Singleton.instance) {
            Singleton.instance = new Singleton();
            // ... any one time initialization goes here ...
        }
        return Singleton.instance;
    }
    someMethod() { }
}

let something = new Singleton() // Error: constructor of 'Singleton' is private.

let instance = Singleton.getInstance() // do something with the instance...
```

Однак, якщо вам не потрібна відкладена ініціалізація, ви можете натомість просто використати `namespace`:
```ts
namespace Singleton {
    // ... any one time initialization goes here ...
    export function someMethod() { }
}
// Usage
Singleton.someMethod();
```

> Попередження: Singleton — це лише дивовижна назва для [global](http://stackoverflow.com/a/142450/390330)

Для більшості проектів `namespace` можна додатково замінити на *module*.

```ts
// someFile.ts
// ... any one time initialization goes here ...
export function someMethod() { }

// Usage
import {someMethod} from "./someFile";
```


