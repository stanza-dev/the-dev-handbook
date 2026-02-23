---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-typeof-patterns"
---

# typeof in Type Context

## Introduction
The `typeof` operator in TypeScript has a dual life: at runtime it returns a string like `"string"` or `"object"`, but in a type position it captures the full static type of a value. This lesson explores the type-level `typeof` and the powerful patterns it enables when combined with `keyof` and indexed access.

## Key Concepts
- **Type-level `typeof`**: Extracts the TypeScript type of a variable, constant, or property for use in type annotations.
- **`as const` assertion**: Narrows a value to its most specific literal type, making `typeof` far more useful.
- **Array element type via `[number]`**: Indexed access with `number` extracts the element type of an array or tuple.

## Real World Context
In real codebases, you frequently define configuration objects, route maps, or constant arrays at runtime and then need type-safe functions that operate on them. Instead of maintaining separate type definitions that drift out of sync, you derive the types directly from the values using `typeof`. This pattern is used heavily in libraries like Zod, tRPC, and Prisma.

## Deep Dive

### Basic typeof in Type Position

When you write `typeof` before a variable name in a type annotation, TypeScript gives you the inferred type of that variable:

```typescript
let message = "hello";
type MessageType = typeof message; // string

const greeting = "hello";
type GreetingType = typeof greeting; // "hello" (literal type because const)
```

Notice the difference: `let` gives a widened type (`string`), while `const` preserves the literal type (`"hello"`).

### typeof with as const

The `as const` assertion makes every property `readonly` and narrows all values to their literal types. This is essential for getting useful types from runtime objects:

```typescript
const COLORS = ["red", "green", "blue"] as const;
type Colors = typeof COLORS; // readonly ["red", "green", "blue"]
type Color = Colors[number];  // "red" | "green" | "blue"
```

Without `as const`, `typeof COLORS` would be `string[]` and `Color` would just be `string` — far less useful.

### Combining keyof and typeof

A common pattern is extracting keys from a runtime object:

```typescript
const STATUS_CODES = {
  OK: 200,
  NOT_FOUND: 404,
  SERVER_ERROR: 500,
} as const;

type StatusName = keyof typeof STATUS_CODES; // "OK" | "NOT_FOUND" | "SERVER_ERROR"
type StatusCode = (typeof STATUS_CODES)[StatusName]; // 200 | 404 | 500
```

The `keyof typeof` idiom first captures the type of the object, then extracts its keys. This is one of the most frequently used patterns in TypeScript.

### Array Element Types with [number]

You can extract the element type from an array type using indexed access with `number`:

```typescript
const PERMISSIONS = ["read", "write", "admin"] as const;
type Permission = (typeof PERMISSIONS)[number]; // "read" | "write" | "admin"
```

This works because arrays have a numeric index signature. The `[number]` index returns the union of all element types.

## Common Pitfalls
1. **Forgetting `as const`** — Without it, `typeof` captures widened types (`string`, `number`) instead of literal types (`"red"`, `200`). Always use `as const` when you want the narrowest possible type.
2. **Using `typeof` on types instead of values** — `typeof` only works on runtime values (variables, properties). Writing `typeof MyInterface` is an error because interfaces do not exist at runtime.

## Best Practices
1. **Define constants once, derive types from them** — The pattern `const X = {...} as const; type X = typeof X;` eliminates duplication and keeps types in sync with values automatically.
2. **Use `satisfies` with `as const` for validation** — In TypeScript 4.9+, `const x = {...} as const satisfies Schema` gives you both literal types and type checking against a schema.

## Summary
- `typeof` in type position captures the static type of a runtime value.
- `as const` narrows values to literal types, making `typeof` much more powerful.
- `keyof typeof obj` extracts the keys of a runtime object as a union of string literals.
- `Array[number]` extracts the element type from an array type.

## Code Examples

**Using typeof with as const and [number] indexed access to derive a union type from a runtime array — a pattern used in most production TypeScript apps**

```typescript
// Deriving a union type from a runtime array
const ROLES = ["admin", "editor", "viewer"] as const;
type Role = (typeof ROLES)[number]; // "admin" | "editor" | "viewer"

// Now you can use Role as a type throughout your app
function hasAccess(userRole: Role, requiredRole: Role): boolean {
  const hierarchy: Record<Role, number> = {
    admin: 3,
    editor: 2,
    viewer: 1,
  };
  return hierarchy[userRole] >= hierarchy[requiredRole];
}

hasAccess("admin", "editor"); // OK
hasAccess("superadmin", "editor"); // Error: '"superadmin"' is not assignable to 'Role'
```


## Resources

- [TypeScript Handbook: Typeof Type Operator](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html) — Official documentation on typeof in type contexts
- [TypeScript Handbook: Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html) — Official documentation on indexed access types including the [number] pattern

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*