---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-distributive-conditional"
---

# Distributive Conditional Types

## Introduction
When a conditional type receives a union as its type argument, something surprising happens: the conditional distributes over each member of the union individually. This behavior — called distributive conditional types — is the mechanism behind `Exclude`, `Extract`, and many other utility types. Understanding distribution is essential for writing correct advanced types.

## Key Concepts
- **Distributive behavior**: `T extends U ? X : Y` applied to `A | B` becomes `(A extends U ? X : Y) | (B extends U ? X : Y)`.
- **Naked type parameter**: Distribution only occurs when the checked type is a "naked" (unwrapped) type parameter — not wrapped in a tuple, array, or other type.
- **Preventing distribution**: Wrap both sides of `extends` in a tuple `[T] extends [U]` to check the union as a whole.

## Real World Context
Distributive conditional types are the engine behind type filtering. When you write `Exclude<'a' | 'b' | 'c', 'a'>` and get `'b' | 'c'`, that works because of distribution. Every time you filter, transform, or partition a union type, you are relying on this behavior.

## Deep Dive

### How Distribution Works

When a conditional type's checked type is a naked type parameter and it receives a union, TypeScript applies the conditional to each union member separately:

```typescript
type ToArray<T> = T extends unknown ? T[] : never;

// With a union:
type Result = ToArray<string | number>;
// Distributes to: (string extends unknown ? string[] : never) | (number extends unknown ? number[] : never)
// Result: string[] | number[]
```

Notice the result is `string[] | number[]` — not `(string | number)[]`. Each member was processed independently.

### Implementing Exclude and Extract

The built-in `Exclude` and `Extract` types are simple conditional types that leverage distribution:

```typescript
// Exclude: remove members assignable to U
type MyExclude<T, U> = T extends U ? never : T;

type A = MyExclude<'a' | 'b' | 'c', 'a'>;
// 'a' extends 'a' ? never : 'a'  =>  never
// 'b' extends 'a' ? never : 'b'  =>  'b'
// 'c' extends 'a' ? never : 'c'  =>  'c'
// Result: 'b' | 'c'

// Extract: keep only members assignable to U
type MyExtract<T, U> = T extends U ? T : never;

type B = MyExtract<string | number | boolean, string | number>;
// Result: string | number
```

Returning `never` from a distributed conditional effectively removes that member from the union, because `never` is the empty type and disappears from unions.

### Naked vs. Wrapped Type Parameters

Distribution only happens when the type parameter is "naked" — used directly without being wrapped:

```typescript
// Naked: T is used directly => distributes
type Naked<T> = T extends string ? "yes" : "no";
type R1 = Naked<string | number>; // "yes" | "no"

// Wrapped: [T] wraps the parameter => does NOT distribute
type Wrapped<T> = [T] extends [string] ? "yes" : "no";
type R2 = Wrapped<string | number>; // "no" (union checked as a whole)
```

Wrapping in a tuple `[T]` on both sides prevents distribution. This is the standard technique when you want to check the entire union rather than its individual members.

### Practical: Filtering Object Keys by Value Type

Distributive conditionals enable filtering object keys based on their value types:

```typescript
type KeysOfType<T, V> = {
  [K in keyof T]: T[K] extends V ? K : never;
}[keyof T];

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

type StringKeys = KeysOfType<User, string>; // "name" | "email"
type NumberKeys = KeysOfType<User, number>; // "id" | "age"
```

This maps each key to itself if its value type matches, or to `never` if it does not. The final `[keyof T]` indexed access collects the non-never results into a union.

## Common Pitfalls
1. **Expecting non-distributive behavior from naked parameters** — If you write `type IsUnion<T> = T extends any ? true : false` and pass a union, each member distributes independently. If you want to check whether `T` itself is a union, you need the wrapped pattern.
2. **Forgetting that `never` is the empty union** — `never` distributes to zero iterations, so `MyExclude<never, string>` is `never`, not `string`. The conditional is never entered.

## Best Practices
1. **Use distribution intentionally** — When you want to transform or filter each member of a union, use a naked type parameter. When you want to examine the union as a whole, wrap it in a tuple.
2. **Return `never` to filter** — In a distributive conditional, returning `never` for unwanted members cleanly removes them from the resulting union.

## Summary
- Distributive conditional types apply the condition to each union member independently.
- Distribution only occurs with naked (unwrapped) type parameters.
- Wrap in `[T] extends [U]` to prevent distribution and check the union as a whole.
- `Exclude` and `Extract` work by distributing over the union and returning `never` for excluded members.

## Code Examples

**Side-by-side comparison of distributive vs. non-distributive conditional types showing how wrapping in a tuple changes the behavior**

```typescript
// Demonstrating distributive vs. non-distributive behavior
type Distributed<T> = T extends string ? T[] : never;
type NonDistributed<T> = [T] extends [string] ? T[] : never;

// Union input: string | number
type A = Distributed<string | number>;
// string[] | never => string[]  (each member checked separately)

type B = NonDistributed<string | number>;
// [string | number] extends [string] => false => never  (checked as a whole)

// This distinction matters when building type utilities
type IsNever<T> = [T] extends [never] ? true : false;
type C = IsNever<never>;  // true  (must use wrapped form!)
type D = IsNever<string>; // false
```


## Resources

- [TypeScript Handbook: Distributive Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types) — Official documentation section on distributive behavior of conditional types

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*