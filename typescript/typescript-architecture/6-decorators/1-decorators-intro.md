---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-decorators-intro"
---

# TC39 Standard Decorators

## Introduction
TypeScript 5.0 introduced support for TC39 standard decorators (Stage 3), a native JavaScript feature for metaprogramming on classes. Unlike the older experimental decorators that required the `experimentalDecorators` flag, standard decorators work out of the box with no configuration — they are simply part of the language.

## Key Concepts
- **Standard Decorator**: A function that receives the decorated value and a context object, and optionally returns a replacement value.
- **Decorator Context**: An object providing metadata about the decoration target (its kind, name, access, and more).
- **No Flag Required**: Standard decorators do not need `experimentalDecorators` in tsconfig.json. They are enabled by default in TypeScript 5.0+.

## Real World Context
Frameworks are migrating to standard decorators: Angular has started adopting them, and new libraries can use decorators without requiring tsconfig flags. Understanding the standard API is essential because the experimental API will eventually be deprecated.

## Deep Dive
### Class Decorators

A standard class decorator receives the class itself and a context object:

```typescript
type ClassDecorator = (
    target: Function,
    context: ClassDecoratorContext
) => Function | void;

function sealed(target: Function, context: ClassDecoratorContext) {
    Object.seal(target);
    Object.seal(target.prototype);
    console.log(`Sealed class: ${String(context.name)}`);
}

@sealed
class Config {
    host = "localhost";
    port = 3000;
}
```

No tsconfig flag is needed. The `context` parameter provides the class name, kind (`"class"`), and metadata.

### How Standard Decorators Differ from Experimental

The key differences:

```typescript
// EXPERIMENTAL (old) — receives constructor only
function oldDecorator(constructor: Function) { /* ... */ }

// STANDARD (new) — receives value + context
function newDecorator(value: Function, context: ClassDecoratorContext) { /* ... */ }
```

Standard decorators receive a `context` object with structured information, while experimental decorators relied on positional arguments that varied by target type.

### Decorator Execution Order

Decorators execute bottom-to-top (innermost first):

```typescript
function first(target: Function, ctx: ClassDecoratorContext) {
    console.log("first");
}

function second(target: Function, ctx: ClassDecoratorContext) {
    console.log("second");
}

@first
@second
class MyClass {}
// Logs: "second" then "first"
```

## Common Pitfalls
1. **Mixing experimental and standard decorators** — If `experimentalDecorators` is enabled in tsconfig, TypeScript uses the old API. To use standard decorators, remove or disable that flag.
2. **Expecting parameter decorators** — Standard decorators do not support parameter decorators (used heavily in NestJS/Angular DI). These frameworks still require `experimentalDecorators` until they migrate.

## Best Practices
1. **Use standard decorators for new projects** — Unless your framework explicitly requires experimental decorators, use the standard API.
2. **Check the context.kind property** — Always verify `context.kind` to ensure your decorator is applied to the correct target type.

## Summary
- TC39 standard decorators are supported in TypeScript 5.0+ with no tsconfig flag.
- They receive `(value, context)` instead of the experimental positional arguments.
- The context object provides name, kind, and metadata about the decorated target.
- Standard decorators do not yet support parameter decorators.

## Code Examples

**A standard class decorator that logs the class name using the context parameter — works in TS 5.0+ without any configuration**

```typescript
// Standard class decorator — no tsconfig flag needed
function log(target: Function, context: ClassDecoratorContext) {
    console.log(`Decorating class: ${String(context.name)}`);
    console.log(`Kind: ${context.kind}`); // "class"
}

@log
class UserService {
    getUser(id: string) { return { id, name: "Alice" }; }
}
// Output: "Decorating class: UserService"
// Output: "Kind: class"
```


## Resources

- [TypeScript 5.0: Decorators](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#decorators) — Official TypeScript 5.0 announcement covering the new standard decorators
- [TC39 Decorators Proposal](https://github.com/tc39/proposal-decorators) — The TC39 Stage 3 decorators proposal that TypeScript implements

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*