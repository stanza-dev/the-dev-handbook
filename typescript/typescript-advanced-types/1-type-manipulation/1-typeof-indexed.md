---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-keyof-typeof-indexed"
---

# Keyof, Typeof, and Indexed Access

## Introduction
TypeScript's type system gives you operators to derive new types from existing ones. The `keyof` operator, `typeof` in type position, and indexed access types form the foundation of type-level programming. Mastering these three tools unlocks the rest of the advanced type system.

## Key Concepts
- **`keyof` operator**: Takes an object type and produces a string or numeric literal union of its keys.
- **`typeof` (type context)**: Captures the type of a runtime value for use in type annotations.
- **Indexed access types**: Uses bracket notation `Type[Key]` to look up the type of a specific property.

## Real World Context
Every time you write a generic function that accepts an object and a key name, you rely on `keyof`. Libraries like Lodash's `_.pick`, React's `setState`, and form libraries like React Hook Form all use these operators internally to provide type-safe APIs. Without them, you would have to resort to `any` or manually duplicate type definitions.

## Deep Dive

### The `keyof` Operator

The `keyof` operator produces a union of literal types representing the keys of an object type. This is one of the most frequently used type operators.

```typescript
type Point = { x: number; y: number };
type P = keyof Point; // "x" | "y"
```

The resulting type `P` is the union `"x" | "y"`. This means a variable of type `P` can only hold one of those two string values.

When the type has a string or number index signature, `keyof` returns those index types:

```typescript
type StringMap = { [key: string]: unknown };
type K = keyof StringMap; // string | number
```

Note that `K` is `string | number` because JavaScript object keys accessed with numbers are coerced to strings, so TypeScript includes both.

### The `typeof` Operator in Type Context

JavaScript already has a `typeof` operator for runtime checks. TypeScript adds a second meaning: when used in a type position, `typeof` extracts the type of a variable or property.

```typescript
const config = { width: 100, height: 200, title: "App" };
type Config = typeof config;
// { width: number; height: number; title: string }
```

This is powerful because you can derive types from existing runtime values instead of maintaining separate type declarations.

### Indexed Access Types

You can look up a specific property's type using bracket notation, just like accessing a property at runtime:

```typescript
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // number
```

You can also use a union to look up multiple properties at once:

```typescript
type AgeOrName = Person["age" | "name"]; // number | string
```

Combining `keyof` with indexed access gives you the union of all property types:

```typescript
type AllValues = Person[keyof Person]; // number | string | boolean
```

This pattern is essential for writing generic utility types.

## Common Pitfalls
1. **Using `keyof` on a value instead of a type** — `keyof myObject` is an error. You need `keyof typeof myObject` to first get the type, then extract the keys.
2. **Forgetting that `keyof` includes index signature types** — If your type has `[key: string]: unknown`, then `keyof` returns `string | number`, not just your named keys.

## Best Practices
1. **Combine `keyof` with generics for safe property access** — The pattern `<T, K extends keyof T>(obj: T, key: K): T[K]` is the foundation of type-safe property accessors.
2. **Prefer `typeof` over manual type duplication** — When you have a runtime constant (config object, array of options), derive the type from it with `typeof` instead of maintaining a parallel type definition.

## Summary
- `keyof T` produces a union of string literal types representing the keys of `T`.
- `typeof value` in type position captures the inferred type of a runtime value.
- `T[K]` performs indexed access, returning the type of property `K` in type `T`.
- These three operators compose together and form the building blocks of advanced type manipulation.

## Code Examples

**Combining keyof, typeof, and indexed access to build a fully type-safe API function from a runtime config object**

```typescript
// keyof + typeof + indexed access working together
const endpoints = {
  users: '/api/users',
  posts: '/api/posts',
  comments: '/api/comments',
} as const;

// Derive types from the runtime value
type Endpoints = typeof endpoints;
type EndpointName = keyof Endpoints; // "users" | "posts" | "comments"
type EndpointPath = Endpoints[EndpointName]; // "/api/users" | "/api/posts" | "/api/comments"

// Type-safe API function
function fetchEndpoint(name: EndpointName): Promise<Response> {
  return fetch(endpoints[name]);
}

fetchEndpoint('users');    // OK
fetchEndpoint('invalid');  // Error: Argument of type '"invalid"' is not assignable
```


## Resources

- [TypeScript Handbook: Keyof Type Operator](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html) — Official documentation on the keyof type operator
- [TypeScript Handbook: Typeof Type Operator](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html) — Official documentation on using typeof in type contexts

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*