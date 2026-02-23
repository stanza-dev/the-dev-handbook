---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-mapped-conditional-combo"
---

# Mapped + Conditional Types

## Introduction
The real power of the TypeScript type system emerges when you combine mapped types with conditional types. This combination lets you selectively transform properties based on their types, filter objects to include only certain kinds of properties, and build sophisticated type utilities that adapt to any input shape.

## Key Concepts
- **Conditional value transformation**: Using `T[K] extends SomeType ? NewType : T[K]` in the value position of a mapped type to selectively transform properties.
- **Type-based key filtering**: Using conditional types in the `as` clause to include or exclude properties based on their value types.
- **Property partitioning**: Splitting an object type into two complementary subsets based on property types.

## Real World Context
This pattern is used to build form validation types (extracting only the fields that need validation), serialization types (converting Date properties to strings), API response transformers (making certain property types nullable for partial loading), and database model types (separating computed fields from stored fields).

## Deep Dive

### Conditional Value Transformation

You can transform property values based on their types:

```typescript
// Convert all Date properties to string (for JSON serialization)
type Serialize<T> = {
  [K in keyof T]: T[K] extends Date ? string : T[K];
};

interface User {
  id: number;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

type SerializedUser = Serialize<User>;
// { id: number; name: string; createdAt: string; updatedAt: string }
```

The conditional checks each property's type: if it extends `Date`, the output type becomes `string`; otherwise the original type is preserved.

### Filtering Properties by Type

Combining the `as` clause with conditionals filters properties:

```typescript
// Extract only function-valued properties
type Methods<T> = {
  [K in keyof T as T[K] extends (...args: any[]) => any ? K : never]: T[K];
};

// Extract only non-function (data) properties
type DataProps<T> = {
  [K in keyof T as T[K] extends (...args: any[]) => any ? never : K]: T[K];
};

class UserService {
  name: string = "";
  email: string = "";
  save(): Promise<void> { return Promise.resolve(); }
  validate(): boolean { return true; }
}

type UserMethods = Methods<UserService>;
// { save: () => Promise<void>; validate: () => boolean }

type UserData = DataProps<UserService>;
// { name: string; email: string }
```

The two types are complementary — together they partition the original type into methods and data properties.

### Making Specific Property Types Optional

A more nuanced transformation: make only nullable properties optional:

```typescript
type NullableToOptional<T> = {
  [K in keyof T as null extends T[K] ? K : never]?: Exclude<T[K], null>;
} & {
  [K in keyof T as null extends T[K] ? never : K]: T[K];
};

interface APIResponse {
  id: number;
  name: string;
  avatar: string | null;
  bio: string | null;
}

type CleanResponse = NullableToOptional<APIResponse>;
// { id: number; name: string; avatar?: string; bio?: string }
```

This splits the object into two parts: nullable properties become optional (with null removed from the type), and non-nullable properties remain required. The intersection (`&`) merges them back into a single type.

### Recursive Mapped Types

You can apply transformations recursively to handle nested objects:

```typescript
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object
    ? T[K] extends Function
      ? T[K]
      : DeepReadonly<T[K]>
    : T[K];
};

interface Config {
  database: {
    host: string;
    port: number;
    credentials: {
      user: string;
      password: string;
    };
  };
}

type FrozenConfig = DeepReadonly<Config>;
// All properties at every level are readonly
```

The recursive call checks if a property is an object (but not a function), and if so, applies `DeepReadonly` recursively.

## Common Pitfalls
1. **Forgetting that intersection types may not simplify** — When you split and rejoin with `&`, the resulting type is technically an intersection. Hovering over it in an IDE may show the full intersection instead of a clean object. Use a helper like `type Simplify<T> = { [K in keyof T]: T[K] }` to flatten it.
2. **Infinite recursion in recursive mapped types** — Without a base case, recursive types can cause compiler errors. Always check for primitive types or `Function` before recursing.

## Best Practices
1. **Use the `as` clause for filtering instead of `Pick`/`Omit` chains** — A single mapped type with `as` is more readable and maintainable than `Pick<T, KeysMatching<T, string>>` composed from multiple utility types.
2. **Create reusable generic filter types** — Build `PropertiesOfType<T, V>` once and reuse it across your codebase instead of inline conditional-mapped types.

## Summary
- Conditional types in the value position transform property types selectively.
- Conditional types in the `as` clause filter properties by their value types.
- Intersection of complementary mapped types partitions an object type.
- Recursive mapped types handle nested object transformations.

## Code Examples

**A reusable generic type that filters an object to include only properties whose values extend a given type — useful for form handling, validation, and serialization**

```typescript
// Generic utility: extract properties of a specific type
type PropertiesOfType<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K];
};

interface FormState {
  username: string;
  age: number;
  email: string;
  termsAccepted: boolean;
  score: number;
}

type StringFields = PropertiesOfType<FormState, string>;
// { username: string; email: string }

type NumberFields = PropertiesOfType<FormState, number>;
// { age: number; score: number }

type BooleanFields = PropertiesOfType<FormState, boolean>;
// { termsAccepted: boolean }
```


## Resources

- [TypeScript Handbook: Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html) — Official documentation on mapped types including conditional transformations

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*