## Advanced mixed-ins in TypeScript.

### What Are Mix-Ins

> In object-oriented programming languages, a Mixin is a class that contains methods for use by other classes without having to be the parent class of those other classes. How those other classes gain access to the mixin's methods depends on the language. Mixins are sometimes described as being "included" rather than "inherited".

-   https://en.wikipedia.org/wiki/Mixin

### The official recommended TypeScript pattern.

-   https://www.typescriptlang.org/docs/handbook/mixins.html

### Issues with the recommended TypeScript pattern.

-   Duplicate declarations of the instance methods.
-   Missing scenario of interaction between different mixins included in the same class.
    -   This scenario is more similar to partial classes.
-   Instance fields initialization would not be executed.
    -   As instance fields are copied to the constructor by the TypeScript compiler
        and only one constructor would get invoked.

### A different TypeScript pattern.

The building blocks are as follows:

-   Use Intersection Types to define the complete Type (after all the mixins).
-   Specify the type of "this" context in methods as the intersected mixed Type
    to allow "interaction" between different mixed-ins of the same class.
-   Create init methods for each mixin to allow instance members initialization.
-   Use a type assertion in the main class constructor to enable calling this init methods.
-   Use an abstraction (factory) to create instances of the mixed class in order to enforce type assertion.

```typescript
// Type Intersection
type SmartObjectMixed = Disposable & Activatable & SmartObject

// Disposable Mixin
class Disposable {
    isDisposed: boolean

    initDisposable() {
        this.isDisposed = false
    }

    dispose() {
        this.isDisposed = true
    }
}

// Activatable Mixin
class Activatable {
    isActive: boolean

    initActivatable() {
        this.isActive = false
    }

    activate() {
        this.isActive = true
    }

    deactivate(this: SmartObjectMixed) {
        this.isActive = false
        // accessing method from another mixin
        this.dispose()
    }
}

class SmartObject {
    constructor() {
        // casting to "that" as constructors do not allow re-defining this "this" context.
        const that: SmartObjectMixed = this as any
        that.initActivatable()
        that.initDisposable()
        setInterval(
            () => console.log(that.isActive + " : " + that.isDisposed),
            500
        )
    }

    interact(this: SmartObjectMixed) {
        this.activate()
    }
}
applyMixins(SmartObject, [Disposable, Activatable])

// An abstraction is required to get around the fact "SmartObject" does not directly
function createSmartObject(): SmartObjectMixed {
    return new SmartObject() as any
}

// Using factory to construct the object as "forced" cast is needed.
const smartObj = createSmartObject()
setTimeout(() => smartObj.interact(), 1000)
```

-   Pros

    -   Avoid duplication.
    -   Allows splitting up large classes to multiple files if/when composition is not appropriate.
        -   a.k.a "partial classes".

-   Cons
    -   Breaks the semantics of TypeScript a bit, What the compiler knows about "SmartObject"
        is no longer the "full story", that is why the factory is needed to create new instances.
