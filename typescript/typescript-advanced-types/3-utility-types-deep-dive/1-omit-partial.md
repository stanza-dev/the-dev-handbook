---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-pick-omit-partial"
---

# Partial, Required, Readonly, Pick, Omit, and Record

## Introduction
TypeScript ships with a set of built-in utility types that perform common type transformations. These types are not magic — they are implemented using mapped types and conditional types that you have already learned. Understanding them deeply lets you use them correctly and compose them for complex transformations.

## Key Concepts
- **`Partial<T>`**: Makes all properties of `T` optional.
- **`Required<T>`**: Makes all properties of `T` required (removes `?`).
- **`Readonly<T>`**: Makes all properties of `T` read-only.
- **`Pick<T, K>`**: Creates a type with only the specified keys `K` from `T`.
- **`Omit<T, K>`**: Creates a type with all keys from `T` except those in `K`.
- **`Record<K, V>`**: Creates an object type with keys of type `K` and values of type `V`.
- **`NonNullable<T>`**: Removes `null` and `undefined` from a union.

## Real World Context
These utility types appear in virtually every TypeScript codebase. `Partial<T>` is used for update operations (PATCH endpoints), `Pick<T, K>` for selecting fields in API responses, `Readonly<T>` for immutable state, and `Record<K, V>` for lookup tables. Knowing which utility to reach for saves you from writing custom mapped types for common patterns.

## Deep Dive

### Partial<T>

`Partial` makes every property optional. Under the hood, it is a mapped type:

```typescript
// Built-in implementation:
type Partial<T> = { [P in keyof T]?: T[P] };

// Usage:
interface User { id: number; name: string; email: string; }
type UpdateUser = Partial<User>;
// { id?: number; name?: string; email?: string }

function updateUser(id: number, fields: Partial<User>) {
  // fields.name is string | undefined
}
updateUser(1, { name: "Alice" }); // Only update name
```

This is the standard pattern for PATCH-style update functions where you only provide the fields you want to change.

### Required<T>

`Required` is the opposite of `Partial` — it removes the `?` modifier from all properties:

```typescript
// Built-in implementation:
type Required<T> = { [P in keyof T]-?: T[P] };

interface Config {
  host?: string;
  port?: number;
  debug?: boolean;
}

type ResolvedConfig = Required<Config>;
// { host: string; port: number; debug: boolean }
```

The `-?` modifier removes optionality. This is useful when you have a config type with defaults and want to represent the fully resolved config after defaults are applied.

### Readonly<T>

`Readonly` adds the `readonly` modifier to every property:

```typescript
// Built-in implementation:
type Readonly<T> = { readonly [P in keyof T]: T[P] };

interface State { count: number; items: string[]; }
type FrozenState = Readonly<State>;

const state: FrozenState = { count: 0, items: [] };
state.count = 1;  // Error: Cannot assign to 'count' because it is a read-only property
state.items.push("a"); // Warning: this still works! Readonly is shallow.
```

Important: `Readonly` is shallow. It prevents reassignment of properties but does not freeze nested objects or arrays.

### Pick<T, K> and Omit<T, K>

`Pick` selects a subset of properties; `Omit` removes properties:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type PublicUser = Pick<User, "id" | "name" | "email">;
// { id: number; name: string; email: string }

type PublicUser2 = Omit<User, "password">;
// { id: number; name: string; email: string }
```

Both approaches produce the same result here. Use `Pick` when you want to explicitly list what to include; use `Omit` when it is easier to list what to exclude.

### Record<K, V>

`Record` creates an object type from a key type and value type:

```typescript
type PageInfo = { title: string; url: string };
type Page = "home" | "about" | "contact";

const nav: Record<Page, PageInfo> = {
  home: { title: "Home", url: "/" },
  about: { title: "About", url: "/about" },
  contact: { title: "Contact", url: "/contact" },
};
```

This is useful for lookup tables, dictionaries, and configuration maps where you want every key in a union to have an entry.

### NonNullable<T>

`NonNullable` removes `null` and `undefined` from a union type:

```typescript
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>; // string
```

Under the hood, it is implemented as a distributive conditional type: `T extends null | undefined ? never : T`.

## Common Pitfalls
1. **Assuming `Readonly` is deep** — `Readonly<T>` only makes the top-level properties read-only. Nested objects can still be mutated. For deep immutability, you need a custom `DeepReadonly` type or a library like Immer.
2. **Using `Omit` with string literals not in the type** — `Omit<User, "nonexistent">` compiles without error but does nothing. TypeScript does not warn you about omitting keys that do not exist.

## Best Practices
1. **Compose utility types** — `Readonly<Pick<User, "id" | "name">>` creates a read-only view of selected properties. Chain them for precise transformations.
2. **Use `satisfies` with `Record` for exhaustive checking** — `const x = {...} satisfies Record<Page, PageInfo>` ensures every key is present while preserving literal types.

## Summary
- `Partial<T>` makes all properties optional; `Required<T>` makes them required.
- `Readonly<T>` prevents property reassignment (shallow only).
- `Pick<T, K>` selects properties; `Omit<T, K>` removes them.
- `Record<K, V>` creates a typed dictionary from a key union and value type.
- `NonNullable<T>` strips `null` and `undefined` from unions.
- These utility types compose together for complex transformations.

## Code Examples

**Composing Omit, Partial, and Readonly to create precise API types from a single base User interface**

```typescript
// Composing utility types for a real-world API pattern
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// API response: no password, all fields required and read-only
type UserResponse = Readonly<Omit<User, "password">>;

// Update payload: no id/createdAt, all remaining fields optional
type UpdatePayload = Partial<Omit<User, "id" | "createdAt" | "password">>;

// Create payload: no id/createdAt, password required
type CreatePayload = Omit<User, "id" | "createdAt">;

function updateUser(id: number, data: UpdatePayload): UserResponse {
  // data.name is string | undefined
  // data.email is string | undefined
  throw "implement";
}
```


## Resources

- [TypeScript Handbook: Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html) — Official reference for all built-in utility types

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*