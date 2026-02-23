---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-indexed-access-deep"
---

# Advanced Indexed Access

## Introduction
You have seen basic indexed access with `Type["key"]`. This lesson goes deeper: multi-level access, union-based lookups, and the patterns that make indexed access types a cornerstone of advanced type manipulation.

## Key Concepts
- **Multi-level indexed access**: Chaining bracket notations to reach nested property types, e.g. `T["a"]["b"]`.
- **Union indexed access**: Using a union of keys to get a union of property types, e.g. `T["a" | "b"]`.
- **`keyof` + indexed access**: Combining `T[keyof T]` to get the union of all value types in a type.

## Real World Context
When working with deeply nested API responses, database schemas, or state trees, you often need to extract the type of a nested property without manually redeclaring it. Multi-level indexed access lets you do this precisely. Redux selectors, GraphQL code generators, and ORM type helpers all rely on this pattern.

## Deep Dive

### Multi-Level Indexed Access

You can chain indexed access to drill into nested types:

```typescript
type APIResponse = {
  data: {
    user: {
      id: number;
      profile: {
        avatar: string;
        bio: string;
      };
    };
  };
  status: number;
};

type UserProfile = APIResponse["data"]["user"]["profile"];
// { avatar: string; bio: string }

type AvatarType = APIResponse["data"]["user"]["profile"]["avatar"];
// string
```

Each bracket lookup returns the type of that property, which you can then index into further. This is analogous to optional chaining at the value level, but at the type level.

### Union Indexed Access

When you pass a union of keys, TypeScript returns a union of the corresponding property types:

```typescript
type Person = {
  name: string;
  age: number;
  active: boolean;
};

type NameOrAge = Person["name" | "age"]; // string | number
type AllValues = Person[keyof Person];    // string | number | boolean
```

This is extremely useful for constraining function parameters to only accept certain value types from an object.

### Indexed Access on Arrays and Tuples

Arrays and tuples support indexed access too. For arrays, `[number]` gives you the element type. For tuples, numeric literal indices give you specific element types:

```typescript
type Tuple = [string, number, boolean];
type First = Tuple[0];     // string
type Second = Tuple[1];    // number
type AnyElement = Tuple[number]; // string | number | boolean
```

This is the mechanism behind utility types like `Parameters<T>` which extract function parameter types as a tuple and then index into it.

### Combining with Generics

Indexed access types become especially powerful in generic contexts:

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30 };
const name = getProperty(user, "name"); // string
const age = getProperty(user, "age");   // number
```

The return type `T[K]` uses indexed access to precisely track which property type is returned based on the key argument.

## Common Pitfalls
1. **Using dot notation instead of bracket notation** — `Type.property` does not work in TypeScript's type system. You must use `Type["property"]` with string literal keys in brackets.
2. **Indexing with a type that is not a valid key** — `Person[string]` only works if `Person` has an index signature. For specific properties, use string literal types like `Person["name"]`.

## Best Practices
1. **Extract deeply nested types instead of redeclaring them** — If an API response type is already defined, use chained indexed access `Response["data"]["items"][number]` rather than creating a new interface.
2. **Use `T[K]` as the return type in generic property access functions** — This preserves the exact type of the accessed property instead of widening to a union.

## Summary
- Chain bracket notation to access nested property types: `T["a"]["b"]["c"]`.
- Pass a union of keys to get a union of property types: `T["a" | "b"]`.
- Use `T[keyof T]` to get the union of all value types in `T`.
- Indexed access on tuples with numeric literals extracts specific element types.

## Code Examples

**Extracting nested types from a database schema using chained indexed access — this pattern is used by ORMs like Prisma and Drizzle**

```typescript
// Real-world pattern: extracting types from a nested API schema
type Schema = {
  tables: {
    users: { id: number; email: string; role: "admin" | "user" };
    posts: { id: number; title: string; authorId: number };
  };
};

// Extract specific table types
type UsersTable = Schema["tables"]["users"];
// { id: number; email: string; role: "admin" | "user" }

// Extract all table names
type TableName = keyof Schema["tables"]; // "users" | "posts"

// Generic table accessor
function getTable<T extends TableName>(
  db: Schema["tables"],
  table: T
): Schema["tables"][T] {
  return db[table];
}
```


## Resources

- [TypeScript Handbook: Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html) — Official documentation on indexed access types

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*