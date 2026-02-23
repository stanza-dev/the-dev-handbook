---
source_course: "typescript"
source_lesson: "typescript-primitives-arrays-any"
---

# Primitives, Arrays, and Any

## Introduction
TypeScript builds on JavaScript's primitive types and adds powerful array typing and the escape-hatch `any` type. Understanding these foundational types is essential because every TypeScript program uses them extensively.

## Key Concepts
- **Primitive Types**: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, and `bigint` — the same primitives JavaScript has.
- **Array Type**: A collection of values of the same type, written as `type[]` or `Array<type>`.
- **The `any` Type**: A special type that disables type checking for a value. Useful for migration but dangerous in new code.
- **Type Inference**: TypeScript automatically determines the type of a variable from its initializer.

## Real World Context
When you fetch data from an API, you need to tell TypeScript what shape the response has. Without proper typing, you might access a property that does not exist and only discover the bug when a user reports it. Primitives and arrays form the building blocks of every data model.

## Deep Dive

### Primitive Types

TypeScript recognizes all JavaScript primitives:

```typescript
const username: string = "Alice";
const itemCount: number = 42;
const isVerified: boolean = true;
const uniqueId: symbol = Symbol("id");
const largeNumber: bigint = 9007199254740991n;
```

For most of these, type inference makes the annotation optional. Writing `const username = "Alice"` is sufficient because TypeScript infers the type as `string`.

### Arrays

Arrays can be typed in two equivalent ways:

```typescript
const scores: number[] = [98, 85, 92];
const names: Array<string> = ["Alice", "Bob", "Carol"];
```

Both syntaxes are identical in behavior. The `type[]` shorthand is more common in practice. TypeScript ensures every element matches the declared type:

```typescript
const prices: number[] = [9.99, 14.50, 3.25];
// prices.push("free"); // Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

This prevents accidentally mixing types in a collection.

### The `any` Type

The `any` type opts out of all type checking:

```typescript
let data: any = { name: "Alice" };
data = 42;          // No error
data.nonExistent(); // No error at compile time — crashes at runtime!
```

Using `any` defeats the purpose of TypeScript. It is acceptable when migrating a large JavaScript codebase incrementally, but new code should never use it.

### Type Inference

TypeScript is smart enough to infer types from context:

```typescript
let temperature = 72; // inferred as number
const greeting = "Hello"; // inferred as the literal type "Hello"
const flags = [true, false, true]; // inferred as boolean[]
```

Notice that `const` declarations infer literal types (`"Hello"` instead of `string`) because the value cannot change.

## Common Pitfalls
1. **Using `any` to fix type errors quickly** — This silences the compiler but hides real bugs. Use `unknown` instead when the type is genuinely uncertain.
2. **Confusing `number[]` with `[number]`** — `number[]` is an array of numbers. `[number]` is a tuple with exactly one numeric element. They are different types.

## Best Practices
1. **Prefer `unknown` over `any`** — The `unknown` type forces you to narrow before using the value, maintaining type safety.
2. **Enable `noImplicitAny`** — This compiler option (included in strict mode) flags variables that would default to `any` without an explicit annotation.

## Summary
- TypeScript supports all JavaScript primitives with lowercase type names.
- Arrays are typed with `type[]` or `Array<type>` syntax.
- Avoid `any` in new code; use `unknown` when the type is genuinely uncertain.
- Type inference keeps code concise — annotate only where necessary.

## Code Examples

**Type inference works automatically for initialized variables, but explicit annotations are needed for uninitialized declarations**

```typescript
// TypeScript infers types from initializers
const appName = "TaskManager";  // type: "TaskManager" (literal)
let userCount = 0;               // type: number
const features = ["auth", "api"]; // type: string[]

// Explicit annotations needed when declaring without initializing
let pendingTasks: string[];
pendingTasks = ["Review PR", "Deploy staging"];
```


## Resources

- [TypeScript Handbook: Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html) — Official guide to the most common types in TypeScript

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*