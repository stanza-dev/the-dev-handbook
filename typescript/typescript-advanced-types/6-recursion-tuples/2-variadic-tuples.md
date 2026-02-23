---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-variadic-tuples"
---

# Variadic Tuple Types

## Introduction
Variadic tuple types, introduced in TypeScript 4.0, let you spread generic type parameters into tuple positions. This enables type-safe operations like concatenating tuples, prepending/appending elements, and building functions that accept variable-length typed argument lists. They are the tuple equivalent of rest parameters at the type level.

## Key Concepts
- **Spread in tuples**: `[...T]` where `T` is a generic tuple type spreads its elements into the containing tuple.
- **Tuple concatenation**: `[...A, ...B]` concatenates two tuple types.
- **Labeled tuple elements**: TypeScript preserves parameter names in tuple types, improving IDE hints.

## Real World Context
Variadic tuples power type-safe function composition (`pipe`, `compose`), argument forwarding (wrappers, decorators), event emitter APIs (typed `emit` and `on` methods), and builder patterns that accumulate typed arguments. They are the foundation of libraries like ts-toolbelt and type-fest's tuple utilities.

## Deep Dive

### Basic Tuple Spread

You can spread generic tuple types into new tuples:

```typescript
type Strings = [string, string];
type Numbers = [number, number];

type Combined = [...Strings, ...Numbers];
// [string, string, number, number]
```

The spread operator `...` works at the type level just like it does at the value level.

### Generic Tuple Concatenation

Combining spreads with generics creates reusable tuple operations:

```typescript
type Concat<A extends any[], B extends any[]> = [...A, ...B];

type Result = Concat<[1, 2], [3, 4]>;
// [1, 2, 3, 4]
```

This is a type-level implementation of array concatenation.

### Prepend and Append

Adding elements to the start or end of a tuple:

```typescript
type Prepend<T, Tuple extends any[]> = [T, ...Tuple];
type Append<Tuple extends any[], T> = [...Tuple, T];

type A = Prepend<boolean, [string, number]>;
// [boolean, string, number]

type B = Append<[string, number], boolean>;
// [string, number, boolean]
```

These simple types are building blocks for more complex tuple manipulation.

### Extracting Parts with Infer

Variadic tuples work with `infer` to destructure tuples:

```typescript
// Remove the first element
type Tail<T extends any[]> = T extends [any, ...infer Rest] ? Rest : [];

// Remove the last element
type Init<T extends any[]> = T extends [...infer Rest, any] ? Rest : [];

// Get the first element
type Head<T extends any[]> = T extends [infer First, ...any[]] ? First : never;

// Get the last element
type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;

type T = [string, number, boolean];
type A = Tail<T>;  // [number, boolean]
type B = Init<T>;  // [string, number]
type C = Head<T>;  // string
type D = Last<T>;  // boolean
```

The `...infer Rest` pattern captures "the rest of the tuple" as a new tuple type.

### Type-Safe Function Wrappers

Variadic tuples enable functions that forward arguments with full type safety:

```typescript
function withLogging<Args extends any[], R>(
  fn: (...args: Args) => R,
  ...args: Args
): R {
  console.log("Calling with:", args);
  const result = fn(...args);
  console.log("Result:", result);
  return result;
}

function add(a: number, b: number): number {
  return a + b;
}

withLogging(add, 1, 2); // Fully typed: Args = [number, number], R = number
withLogging(add, "1", 2); // Error: string is not assignable to number
```

The `Args extends any[]` generic captures the exact tuple of parameter types, preserving full type checking through the wrapper.

### Recursive Tuple Operations

Combining variadic tuples with recursion enables operations like `Reverse`:

```typescript
type Reverse<T extends any[]> =
  T extends [infer First, ...infer Rest]
    ? [...Reverse<Rest>, First]
    : [];

type Original = [1, 2, 3, 4];
type Reversed = Reverse<Original>; // [4, 3, 2, 1]
```

Each recursion step takes the first element and appends it to the end of the reversed rest.

## Common Pitfalls
1. **Confusing `T[]` with `[...T]`** — `T[]` is an array of elements of type `T`. `[...T]` where `T extends any[]` spreads a tuple type. They are structurally different.
2. **Recursion depth limits on large tuples** — Recursive tuple operations hit TypeScript's depth limit around 25-50 levels. For larger tuples, consider tail-call optimized patterns or avoid recursion.

## Best Practices
1. **Use variadic tuples for type-safe argument forwarding** — Whenever you write a wrapper or decorator function, use `(...args: Args)` with a generic `Args extends any[]` to preserve the wrapped function's signature.
2. **Prefer built-in tuple operations over recursive ones** — TypeScript's built-in spread `[...A, ...B]` is more efficient than custom recursive concatenation.

## Summary
- Variadic tuples use `[...T]` to spread generic tuple types.
- They enable type-safe concatenation, prepend, append, and extraction.
- Combined with `infer`, they destructure tuples into head, tail, and rest.
- They are essential for type-safe function wrappers that forward arguments.

## Code Examples

**A type-safe pipe function using overloads to track the type through each transformation step — variadic tuples make the argument forwarding possible**

```typescript
// Type-safe pipe function using variadic tuples
type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;

function pipe<A, B>(a: A, fn1: (a: A) => B): B;
function pipe<A, B, C>(a: A, fn1: (a: A) => B, fn2: (b: B) => C): C;
function pipe<A, B, C, D>(a: A, fn1: (a: A) => B, fn2: (b: B) => C, fn3: (c: C) => D): D;
function pipe(value: any, ...fns: Function[]): any {
  return fns.reduce((acc, fn) => fn(acc), value);
}

// Fully typed pipeline
const result = pipe(
  "  Hello World  ",
  (s: string) => s.trim(),       // string
  (s: string) => s.toLowerCase(), // string
  (s: string) => s.split(" "),   // string[]
);
// result: string[]
```


## Resources

- [TypeScript 4.0: Variadic Tuple Types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-0.html#variadic-tuple-types) — TypeScript 4.0 release notes introducing variadic tuple types

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*