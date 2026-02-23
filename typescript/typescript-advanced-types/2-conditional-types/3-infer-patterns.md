---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-infer-patterns"
---

# Advanced Infer Patterns

## Introduction
The `infer` keyword is your Swiss army knife for extracting types from complex structures. Beyond the basic `ReturnType` pattern, `infer` can extract promise results, tuple elements, function parameters, constructor types, and more. This lesson explores the most useful `infer` patterns you will encounter in production TypeScript.

## Key Concepts
- **`infer` in function signatures**: Extract return types, parameter types, or `this` types from functions.
- **`infer` in promise unwrapping**: Recursively unwrap `Promise<Promise<T>>` to get the innermost type.
- **`infer` in tuple positions**: Extract the first, last, or rest elements of a tuple.
- **Multiple `infer` in one condition**: You can use several `infer` declarations in a single `extends` clause.

## Real World Context
TypeScript's built-in `ReturnType`, `Parameters`, `ConstructorParameters`, and `Awaited` are all built with `infer`. When you work with higher-order functions, middleware chains, or promise-based APIs, knowing how to extract types with `infer` saves you from manual type declarations that break when the source type changes.

## Deep Dive

### Extracting Function Return Types

The classic `infer` pattern extracts the return type of a function:

```typescript
type MyReturnType<T extends (...args: any) => any> =
  T extends (...args: any) => infer R ? R : never;

type A = MyReturnType<() => string>;          // string
type B = MyReturnType<(x: number) => void>;   // void
```

The `infer R` in the return position tells TypeScript to capture whatever type appears there.

### Extracting Function Parameters

Similarly, you can extract all parameter types as a tuple:

```typescript
type MyParameters<T extends (...args: any) => any> =
  T extends (...args: infer P) => any ? P : never;

type C = MyParameters<(a: string, b: number) => void>;
// [a: string, b: number]
```

The result is a tuple type with labeled elements matching the parameter names.

### Promise Unwrapping

A single level of promise unwrapping is straightforward:

```typescript
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type D = UnwrapPromise<Promise<string>>;         // string
type E = UnwrapPromise<Promise<Promise<number>>>; // Promise<number> (only one level!)
```

For deep unwrapping (like the built-in `Awaited`), you need recursion:

```typescript
type DeepAwaited<T> = T extends Promise<infer U> ? DeepAwaited<U> : T;

type F = DeepAwaited<Promise<Promise<Promise<string>>>>; // string
```

The recursive call keeps unwrapping until the type is no longer a `Promise`.

### Tuple Element Extraction

`infer` combined with variadic tuples lets you extract parts of a tuple:

```typescript
// First element
type Head<T extends any[]> = T extends [infer First, ...any[]] ? First : never;

// Last element
type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;

// All elements except the first
type Tail<T extends any[]> = T extends [any, ...infer Rest] ? Rest : never;

type G = Head<[string, number, boolean]>;  // string
type H = Last<[string, number, boolean]>;  // boolean
type I = Tail<[string, number, boolean]>;  // [number, boolean]
```

These patterns are the building blocks for type-level list operations.

### Multiple Infer Declarations

You can use multiple `infer` keywords in a single conditional:

```typescript
type FirstAndLast<T extends any[]> =
  T extends [infer F, ...any[], infer L] ? [F, L] : never;

type J = FirstAndLast<[1, 2, 3, 4]>; // [1, 4]
```

TypeScript infers each variable independently based on its position in the pattern.

## Common Pitfalls
1. **Expecting deep unwrapping from a single infer** — `Promise<infer U>` only unwraps one layer. For nested promises, you need recursion or the built-in `Awaited` type.
2. **Forgetting that infer only works inside extends** — You cannot write `type X = infer T`. The `infer` keyword is only valid in the condition part of a conditional type.

## Best Practices
1. **Constrain the generic before inferring** — Use `T extends (...args: any) => any` to ensure `T` is a function before trying to infer its parts. This gives better error messages when the type does not match.
2. **Prefer built-in utility types when available** — Use `ReturnType<T>`, `Parameters<T>`, and `Awaited<T>` instead of reimplementing them. Write custom `infer` patterns only for shapes the built-ins do not cover.

## Summary
- `infer` captures type variables from within `extends` clauses.
- Use it to extract return types, parameter types, promise contents, and tuple elements.
- For deep unwrapping, combine `infer` with recursion.
- Multiple `infer` declarations in one condition extract multiple parts simultaneously.

## Code Examples

**AsyncReturnType combines typeof, infer, and conditional types to extract the resolved return type of any async function**

```typescript
// Extract the resolved type of an async function's return value
type AsyncReturnType<T extends (...args: any) => Promise<any>> =
  T extends (...args: any) => Promise<infer R> ? R : never;

// Usage with real async functions
async function getUser(id: number) {
  return { id, name: "Alice", role: "admin" as const };
}

async function getPosts() {
  return [{ id: 1, title: "Hello" }, { id: 2, title: "World" }];
}

type User = AsyncReturnType<typeof getUser>;
// { id: number; name: string; role: "admin" }

type Posts = AsyncReturnType<typeof getPosts>;
// { id: number; title: string }[]
```


## Resources

- [TypeScript Handbook: Inferring Within Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types) — Official documentation on using infer in conditional types

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*