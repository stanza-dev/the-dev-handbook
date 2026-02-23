---
source_course: "typescript"
source_lesson: "typescript-const-type-parameters"
---

# Const Type Parameters

## Introduction
Introduced in TypeScript 5.0, const type parameters allow you to infer literal types from arguments without requiring callers to use `as const`. This feature makes generic functions more precise and easier to use, particularly when building APIs that work with configuration objects and fixed sets of values.

## Key Concepts
- **Const Type Parameter (`const T`)**: A type parameter prefixed with `const` that infers the narrowest (literal) type from the argument.
- **Literal Inference**: Without `const`, TypeScript widens `"hello"` to `string`. With `const`, it stays as `"hello"`.
- **Immutable Inference**: Const type parameters also make inferred arrays and objects readonly.

## Real World Context
When building type-safe builders, routers, or event systems, you want the inferred types to be as precise as possible. Before TypeScript 5.0, callers had to write `as const` at every call site. Const type parameters move this responsibility to the function author, making the API simpler for consumers.

## Deep Dive

### The Problem: Widening

Without `const`, TypeScript widens literal types:

```typescript
function createRoute<T extends string>(path: T): { path: T } {
  return { path };
}

const route = createRoute("/dashboard");
// route.path is string — not "/dashboard"
```

TypeScript widens `"/dashboard"` to `string` because `T extends string` allows any string. The specific path is lost.

### The Solution: `const` Type Parameter

Prefixing the type parameter with `const` preserves the literal:

```typescript
function createRoute<const T extends string>(path: T): { path: T } {
  return { path };
}

const route = createRoute("/dashboard");
// route.path is "/dashboard" — literal type preserved!
```

The `const` modifier tells TypeScript to infer the narrowest possible type, just as if the caller had written `as const`.

### With Objects and Arrays

Const type parameters work with complex types too:

```typescript
function defineConfig<const T extends Record<string, unknown>>(config: T): T {
  return config;
}

const config = defineConfig({
  port: 3000,
  host: "localhost",
  features: ["auth", "logging"],
});

// config.port is 3000 (literal), not number
// config.host is "localhost" (literal), not string
// config.features is readonly ["auth", "logging"] — narrowed and readonly
```

The entire structure gets literal inference, including nested arrays becoming readonly tuples.

### Practical Example: Type-Safe Event Emitter

```typescript
function on<const TEvent extends string>(
  event: TEvent,
  handler: (event: TEvent) => void
): void {
  // Register handler...
}

on("click", (event) => {
  // event is "click" (literal), not string
  console.log(`Handling: ${event}`);
});
```

The handler receives the exact event name as a literal type, enabling powerful type-level patterns like event-specific payloads.

### Before vs After

The improvement is about developer experience:

```typescript
// Before TypeScript 5.0 — caller must use 'as const'
const route = createRoute("/dashboard" as const);

// After TypeScript 5.0 — function handles it
const route = createRoute("/dashboard"); // Just works
```

The `as const` burden moves from every call site to the single function definition.

## Common Pitfalls
1. **Using `const` when widening is desired** — If you want `T` to be `string` (not a literal), do not use `const`. It is only appropriate when you need the specific value type.
2. **Forgetting that arrays become readonly** — With `const` type parameters, inferred arrays are `readonly`. If the function modifies the array, this causes type errors.

## Best Practices
1. **Use `const` type parameters in builder/factory functions** — Any function that creates configuration, routes, or schemas benefits from literal inference.
2. **Document the literal inference behavior** — Since this is a newer feature, add JSDoc comments explaining that the function preserves literal types.

## Summary
- `const` type parameters (TypeScript 5.0+) infer literal types without requiring `as const` at the call site.
- They make generic APIs more precise and easier to use.
- Arrays inferred through const type parameters become readonly.
- Use them in builder functions, configuration factories, and event systems.

## Code Examples

**The const modifier on the type parameter automatically infers literal types for the method and path properties**

```typescript
// const type parameter preserves literal types automatically
function createEndpoint<const T extends { method: string; path: string }>(
  config: T
): T {
  return config;
}

const endpoint = createEndpoint({
  method: "GET",
  path: "/api/users",
});

// endpoint.method is "GET" (literal), not string
// endpoint.path is "/api/users" (literal), not string
// No 'as const' needed at the call site!
```


## Resources

- [TypeScript 5.0: Const Type Parameters](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html#const-type-parameters) — Official release notes for the const type parameter feature

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*