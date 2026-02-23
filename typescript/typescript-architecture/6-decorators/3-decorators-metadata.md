---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-decorators-metadata"
---

# Decorator Metadata & Legacy Comparison

## Introduction
TC39 standard decorators include a built-in metadata system via `context.metadata`, eliminating the need for the `reflect-metadata` polyfill. This lesson covers the metadata API and compares standard decorators with the legacy experimental API to help you understand which projects use which and how to migrate.

## Key Concepts
- **Decorator Metadata**: An object attached to `context.metadata` that decorators can write to and read from at runtime.
- **Symbol.metadata**: The well-known symbol used to access metadata on a class after decoration.
- **Legacy Decorators**: The experimental decorator API (`experimentalDecorators: true`) using positional parameters and `reflect-metadata`.

## Real World Context
NestJS, Angular, and TypeORM currently use legacy decorators with `reflect-metadata` for dependency injection and schema definition. As these frameworks migrate to standard decorators, understanding both APIs helps you work with existing codebases and prepare for the transition.

## Deep Dive
### Standard Decorator Metadata

Every decorator context has a `metadata` property — a plain object shared across all decorators on a class:

```typescript
function route(path: string) {
    return function (target: Function, context: ClassMethodDecoratorContext) {
        // Write to the shared metadata object
        (context.metadata.routes ??= []) as string[];
        (context.metadata.routes as string[]).push(
            `${String(context.name)}: ${path}`
        );
        return target;
    };
}

class Controller {
    @route("/users")
    getUsers() {}

    @route("/users/:id")
    getUser() {}
}

// Read metadata via Symbol.metadata
const meta = (Controller as any)[Symbol.metadata];
console.log(meta.routes);
// ["getUsers: /users", "getUser: /users/:id"]
```

No `reflect-metadata` import needed. The metadata object is plain JavaScript.

### Legacy vs Standard Comparison

| Feature | Legacy (experimental) | Standard (TC39) |
|---------|----------------------|-----------------|
| tsconfig flag | `experimentalDecorators: true` | None (default) |
| Class decorator args | `(constructor)` | `(value, context)` |
| Method decorator args | `(target, key, descriptor)` | `(value, context)` |
| Parameter decorators | Supported | Not supported |
| Metadata | `reflect-metadata` polyfill | `context.metadata` built-in |
| Status | Will be deprecated | Future standard |

### Migration Tips

```typescript
// LEGACY method decorator
function legacyLog(target: any, key: string, desc: PropertyDescriptor) {
    const original = desc.value;
    desc.value = function (...args: any[]) {
        console.log(`Calling ${key}`);
        return original.apply(this, args);
    };
}

// STANDARD equivalent
function standardLog(target: Function, context: ClassMethodDecoratorContext) {
    return function (this: any, ...args: any[]) {
        console.log(`Calling ${String(context.name)}`);
        return target.call(this, ...args);
    };
}
```

## Common Pitfalls
1. **Enabling both experimental and standard** — If `experimentalDecorators` is true, TypeScript interprets all decorators as legacy. You cannot mix both APIs in the same project.
2. **Relying on parameter decorators** — Standard decorators do not support parameter decoration. Frameworks using DI (NestJS, Angular) still need the experimental flag until they provide alternatives.

## Best Practices
1. **New projects: use standard** — If your framework supports it, prefer standard decorators for future-proofing.
2. **Existing projects: wait for framework migration** — Do not remove `experimentalDecorators` until your framework officially supports standard decorators.

## Summary
- Standard decorators have built-in metadata via `context.metadata` — no polyfill needed.
- Access metadata on decorated classes via `Symbol.metadata`.
- Legacy decorators use `(target, key, descriptor)` while standard use `(value, context)`.
- Standard decorators do not support parameter decorators yet.

## Code Examples

**Using context.metadata to mark a class as injectable — the metadata is readable at runtime via Symbol.metadata**

```typescript
// Standard decorator with metadata
function injectable(target: Function, context: ClassDecoratorContext) {
    context.metadata.injectable = true;
    context.metadata.className = String(context.name);
}

@injectable
class UserService {
    getUsers() { return []; }
}

// Reading metadata — no reflect-metadata needed
const meta = (UserService as any)[Symbol.metadata];
console.log(meta.injectable); // true
console.log(meta.className);  // "UserService"
```


## Resources

- [TC39 Decorator Metadata Proposal](https://github.com/tc39/proposal-decorator-metadata) — The TC39 proposal for decorator metadata that TypeScript implements

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*