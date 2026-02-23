---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-conditional-types-basics"
---

# Conditional Types

## Introduction
Conditional types bring `if/else` logic to the type system. They let you choose between two types based on a condition, enabling types that adapt to their inputs. This is one of the most powerful features in TypeScript and the foundation for many built-in utility types.

## Key Concepts
- **Conditional type syntax**: `T extends U ? X : Y` — if `T` is assignable to `U`, the type resolves to `X`; otherwise it resolves to `Y`.
- **Distributive conditional types**: When `T` is a union, the conditional distributes over each member independently.
- **The `infer` keyword**: Declares a type variable within an `extends` clause that TypeScript will infer from context.

## Real World Context
Conditional types power the built-in utility types you use daily: `Exclude`, `Extract`, `ReturnType`, `Parameters`, and `Awaited` are all implemented with conditional types. Understanding how they work lets you build your own type utilities for API response unwrapping, form validation typing, and state management patterns.

## Deep Dive

### Basic Syntax

A conditional type checks whether a type relationship holds:

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false
type C = IsString<"hello">; // true (string literal extends string)
```

The `extends` keyword here means "is assignable to". If `T` can be assigned to `string`, the type resolves to `true`; otherwise `false`.

### Conditional Types with Generics

Conditional types become powerful when combined with generics. You can create types that adapt their output based on input:

```typescript
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;

interface IdLabel { id: number; }
interface NameLabel { name: string; }

function createLabel<T extends number | string>(value: T): NameOrId<T> {
  throw "unimplemented";
}

const idLabel = createLabel(42);      // IdLabel
const nameLabel = createLabel("ts");  // NameLabel
```

The return type narrows automatically based on the argument type.

### The `infer` Keyword

Within the `extends` clause, `infer` declares a type variable that TypeScript fills in:

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type A = ReturnType<() => string>;           // string
type B = ReturnType<(x: number) => boolean>; // boolean
type C = ReturnType<string>;                 // never (not a function)
```

Here, `infer R` tells TypeScript: "if `T` is a function type, capture its return type as `R` and use it." If `T` is not a function, the condition fails and we get `never`.

### Nested Conditionals

You can chain conditional types for multi-branch logic:

```typescript
type TypeName<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  "object";

type A = TypeName<string>;    // "string"
type B = TypeName<boolean[]>; // "object"
```

Each branch is checked in order, similar to an if/else-if chain.

## Common Pitfalls
1. **Forgetting about distribution over unions** — `IsString<string | number>` does not check the union as a whole. Instead, it distributes: `IsString<string> | IsString<number>` which gives `true | false`. This is covered in detail in the next lesson.
2. **Using `infer` outside of `extends`** — The `infer` keyword can only appear within the condition part of a conditional type (the `extends` clause). Using it elsewhere is a syntax error.

## Best Practices
1. **Use conditional types to narrow return types** — When a function can return different types based on its input, use a conditional type on the generic parameter to get precise return types.
2. **Keep conditional types shallow when possible** — Deeply nested conditionals become hard to read. Extract intermediate types with meaningful names.

## Summary
- Conditional types use `T extends U ? X : Y` to choose types based on assignability.
- The `infer` keyword captures parts of a type for use in the true branch.
- Conditional types distribute over unions by default.
- Most built-in utility types (`ReturnType`, `Exclude`, `Extract`) are conditional types under the hood.

## Code Examples

**UnwrapPromise utility type using infer to extract the resolved type from a Promise, and leaving non-Promise types unchanged**

```typescript
// Unwrap a Promise type, or leave it as-is
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type A = UnwrapPromise<Promise<string>>;  // string
type B = UnwrapPromise<Promise<number>>;  // number
type C = UnwrapPromise<boolean>;          // boolean (not a Promise, returned as-is)

// Practical use: extracting the resolved type from an async function
async function fetchUser() { return { id: 1, name: "Alice" }; }
type User = UnwrapPromise<ReturnType<typeof fetchUser>>;
// { id: number; name: string }
```


## Resources

- [TypeScript Handbook: Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html) — Official documentation on conditional types including infer and distribution

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*