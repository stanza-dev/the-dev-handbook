---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-decorators-method-field"
---

# Method & Field Decorators

## Introduction
Beyond class decorators, the TC39 standard supports decorating methods, getters, setters, and class fields. Each decorator target receives a different context type, allowing you to intercept method calls, validate field assignments, and add computed behavior.

## Key Concepts
- **Method Decorator**: Receives the method function and a `ClassMethodDecoratorContext`. Can return a replacement function.
- **Field Decorator (Auto-Accessor)**: Uses the `accessor` keyword for decorated fields. Receives an object with `get` and `set` and a `ClassAccessorDecoratorContext`.
- **Getter/Setter Decorator**: Decorates `get` or `set` accessors with their respective context types.

## Real World Context
Method decorators are used for logging, caching, rate-limiting, and authorization. Field decorators with `accessor` enable validation-on-assignment and observable properties.

## Deep Dive
### Method Decorators

A method decorator wraps the original method:

```typescript
function logged(
    target: Function,
    context: ClassMethodDecoratorContext
) {
    const methodName = String(context.name);
    return function (this: any, ...args: any[]) {
        console.log(`Calling ${methodName} with`, args);
        const result = target.call(this, ...args);
        console.log(`${methodName} returned`, result);
        return result;
    };
}

class Calculator {
    @logged
    add(a: number, b: number): number {
        return a + b;
    }
}

const calc = new Calculator();
calc.add(2, 3);
// "Calling add with [2, 3]"
// "add returned 5"
```

The decorator returns a new function that wraps the original, adding logging before and after.

### Auto-Accessor Decorators

The `accessor` keyword creates a field with getter/setter that decorators can intercept:

```typescript
function clamp(min: number, max: number) {
    return function (
        target: ClassAccessorDecoratorTarget<any, number>,
        context: ClassAccessorDecoratorContext<any, number>
    ): ClassAccessorDecoratorResult<any, number> {
        return {
            set(value: number) {
                target.set.call(this, Math.min(max, Math.max(min, value)));
            },
            get() {
                return target.get.call(this);
            },
            init(value: number) {
                return Math.min(max, Math.max(min, value));
            }
        };
    };
}

class Slider {
    @clamp(0, 100)
    accessor value = 50;
}

const slider = new Slider();
slider.value = 150; // Clamped to 100
slider.value = -10; // Clamped to 0
```

### Decorator Factories

Most real-world decorators are factories — functions that return a decorator:

```typescript
function throttle(ms: number) {
    return function (target: Function, context: ClassMethodDecoratorContext) {
        let lastCall = 0;
        return function (this: any, ...args: any[]) {
            const now = Date.now();
            if (now - lastCall < ms) return;
            lastCall = now;
            return target.call(this, ...args);
        };
    };
}

class SearchBox {
    @throttle(300)
    search(query: string) { /* API call */ }
}
```

## Common Pitfalls
1. **Forgetting `accessor` for field decorators** — Standard decorators on plain class fields are limited. Use `accessor` to get full get/set interception.
2. **Losing `this` context** — When wrapping methods, use a regular function (not arrow) and call `target.call(this, ...args)` to preserve the instance context.

## Best Practices
1. **Use decorator factories for configurability** — `@throttle(300)` is more flexible than a fixed `@throttle`.
2. **Type the context parameter** — Use the specific context types (`ClassMethodDecoratorContext`, `ClassAccessorDecoratorContext`) for maximum type safety.

## Summary
- Method decorators receive the function and return a replacement wrapper.
- Auto-accessor decorators use the `accessor` keyword for field get/set interception.
- Decorator factories accept configuration and return the actual decorator.
- Always preserve `this` context when wrapping methods.

## Code Examples

**A memoize method decorator that caches results by argument — subsequent calls with the same args return instantly**

```typescript
// Method decorator factory for memoization
function memoize(
    target: Function,
    context: ClassMethodDecoratorContext
) {
    const cache = new Map<string, unknown>();
    return function (this: any, ...args: any[]) {
        const key = JSON.stringify(args);
        if (cache.has(key)) return cache.get(key);
        const result = target.call(this, ...args);
        cache.set(key, result);
        return result;
    };
}

class MathService {
    @memoize
    fibonacci(n: number): number {
        if (n <= 1) return n;
        return this.fibonacci(n - 1) + this.fibonacci(n - 2);
    }
}

const math = new MathService();
math.fibonacci(40); // Fast after first call due to caching
```


## Resources

- [TypeScript Handbook: Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html) — Official TypeScript documentation on decorators

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*