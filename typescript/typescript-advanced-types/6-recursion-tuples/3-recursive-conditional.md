---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-recursive-conditional"
---

# Recursive Conditional Types

## Introduction
Recursive conditional types combine the power of conditional types with recursion to perform deep transformations on types. They can traverse nested object structures, unwrap layered generics, flatten deeply nested types, and compute type-level results that would be impossible with a single pass. TypeScript 4.1+ supports recursive conditional types, unlocking a new level of type-level programming.

## Key Concepts
- **Recursive conditional type**: A conditional type whose true or false branch references itself.
- **DeepReadonly**: A recursive mapped type that makes all properties at all levels read-only.
- **DeepPartial**: A recursive mapped type that makes all properties at all levels optional.
- **Flatten**: A recursive type that reduces nested arrays to a single level.

## Real World Context
Recursive conditional types are used in production for deep immutability (state management), deep partial updates (form state), type-safe path expressions (nested object access like `lodash.get`), and schema validation (deeply typed validation rules). Libraries like Zod, tRPC, and Prisma use recursive types extensively.

## Deep Dive

### DeepReadonly

The built-in `Readonly<T>` is shallow. A deep version recurses into nested objects:

```typescript
type DeepReadonly<T> = T extends Function
  ? T
  : T extends object
    ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
    : T;

interface AppState {
  user: {
    name: string;
    preferences: {
      theme: "light" | "dark";
      notifications: boolean;
    };
  };
  posts: { id: number; title: string }[];
}

type FrozenState = DeepReadonly<AppState>;
// All properties at every level are readonly
// state.user.preferences.theme = "dark"; // Error!
```

The recursion checks: if the property is a function, keep it as-is. If it is an object, recurse. Otherwise (primitives), return the type unchanged.

### DeepPartial

Make every property at every depth optional:

```typescript
type DeepPartial<T> = T extends Function
  ? T
  : T extends object
    ? { [K in keyof T]?: DeepPartial<T[K]> }
    : T;

type PartialState = DeepPartial<AppState>;
// Every property at every level is optional

function updateState(patch: DeepPartial<AppState>) {
  // Can pass any subset of the state tree
}

updateState({ user: { preferences: { theme: "dark" } } }); // Valid!
```

This is commonly used for deep merge / deep update patterns.

### Flatten (Recursive Array Unwrapping)

Flatten a nested array type to a single-level array:

```typescript
type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;

type A = Flatten<number[][][]>;  // number
type B = Flatten<string[]>;      // string
type C = Flatten<boolean>;       // boolean (not an array, returned as-is)
```

Each recursion step unwraps one array level. The recursion stops when the type is no longer an array.

### Type-Level String Operations

Recursive conditionals work with template literals for string processing:

```typescript
// Split a string type by a delimiter
type Split<S extends string, D extends string> =
  S extends `${infer Head}${D}${infer Tail}`
    ? [Head, ...Split<Tail, D>]
    : [S];

type Parts = Split<"a.b.c.d", ".">;
// ["a", "b", "c", "d"]

type PathParts = Split<"users/123/posts", "/">;
// ["users", "123", "posts"]
```

The recursion matches the delimiter in the string, splits off the head, and continues with the tail.

### Type-Safe Deep Property Access

Combining recursive types with template literals enables typed path access:

```typescript
type Get<T, Path extends string> =
  Path extends `${infer Key}.${infer Rest}`
    ? Key extends keyof T
      ? Get<T[Key], Rest>
      : never
    : Path extends keyof T
      ? T[Path]
      : never;

interface Config {
  database: {
    host: string;
    port: number;
    auth: {
      username: string;
      password: string;
    };
  };
}

type Host = Get<Config, "database.host">; // string
type User = Get<Config, "database.auth.username">; // string
type Bad = Get<Config, "database.missing">; // never
```

This type recursively follows each segment of the dot-separated path through the type structure.

## Common Pitfalls
1. **Hitting the recursion depth limit** — TypeScript limits recursive type instantiation to roughly 50 levels. Types that process long tuples or deeply nested objects may hit this limit. Consider tail-call optimization patterns where possible.
2. **Forgetting to handle functions** — In `DeepReadonly` and `DeepPartial`, if you do not check for `Function` before checking for `object`, functions (which are objects) will be incorrectly transformed.

## Best Practices
1. **Always check for primitives and functions before recursing** — The standard pattern is: `T extends Function ? T : T extends object ? Recurse<T> : T`.
2. **Use recursive conditional types sparingly** — They are powerful but can slow down the compiler and produce hard-to-read error messages. Prefer shallow utility types when deep recursion is not needed.

## Summary
- Recursive conditional types combine conditional logic with self-reference for deep transformations.
- `DeepReadonly` and `DeepPartial` are the most common recursive type utilities.
- `Flatten` recursively unwraps nested array types.
- Template literal + recursive conditional types enable type-safe path access and string parsing.
- Always handle the base cases (primitives, functions) before recursing into objects.

## Code Examples

**DeepRequired recursively removes optional modifiers at every level — useful for representing configuration after all defaults have been applied**

```typescript
// DeepRequired: the opposite of DeepPartial
type DeepRequired<T> = T extends Function
  ? T
  : T extends object
    ? { [K in keyof T]-?: DeepRequired<T[K]> }
    : T;

interface FormConfig {
  fields?: {
    username?: { required?: boolean; minLength?: number };
    email?: { required?: boolean; pattern?: string };
  };
  submission?: {
    url?: string;
    method?: "GET" | "POST";
  };
}

// After resolving all defaults, everything is required
type ResolvedConfig = DeepRequired<FormConfig>;
// {
//   fields: {
//     username: { required: boolean; minLength: number };
//     email: { required: boolean; pattern: string };
//   };
//   submission: {
//     url: string;
//     method: "GET" | "POST";
//   };
// }
```


## Resources

- [TypeScript 4.1: Recursive Conditional Types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#recursive-conditional-types) — TypeScript 4.1 release notes introducing recursive conditional types

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*