---
source_course: "typescript"
source_lesson: "typescript-generic-constraints"
---

# Generic Constraints

## Introduction
Sometimes a generic function needs to access specific properties on its type parameter. Without constraints, `T` could be anything, so TypeScript cannot guarantee any properties exist. Generic constraints use the `extends` keyword to limit what types are acceptable, giving you access to their known properties.

## Key Concepts
- **Generic Constraint (`extends`)**: Limits a type parameter to types that match a specific shape.
- **Minimum Shape**: The constraint defines the minimum set of properties a type must have.
- **Default Type Parameters**: A fallback type used when the caller does not specify a type argument.

## Real World Context
A generic `sortBy` function needs to know that items have a comparable property. A `merge` function needs to know its arguments are objects. Constraints encode these requirements so the compiler enforces them and the function body can safely access the required properties.

## Deep Dive

### The Problem Without Constraints

Without a constraint, accessing properties on `T` is an error:

```typescript
function getLength<T>(value: T): number {
  return value.length; // Error: Property 'length' does not exist on type 'T'
}
```

TypeScript does not know that `T` has a `length` property. It could be `number`, which has no `length`.

### Adding a Constraint

Use `extends` to require a minimum shape:

```typescript
function getLength<T extends { length: number }>(value: T): number {
  return value.length; // OK — T is guaranteed to have .length
}

getLength("hello");    // OK — string has .length
getLength([1, 2, 3]);  // OK — array has .length
// getLength(42);      // Error — number has no .length
```

The constraint `{ length: number }` means: any type is accepted as long as it has a numeric `length` property.

### Constraining with Interfaces

For complex constraints, use named interfaces:

```typescript
interface HasId {
  id: string;
}

function findById<T extends HasId>(items: T[], targetId: string): T | undefined {
  return items.find(item => item.id === targetId);
}

const users = [
  { id: "u1", name: "Alice", role: "admin" },
  { id: "u2", name: "Bob", role: "user" },
];

const found = findById(users, "u1");
// found is { id: string; name: string; role: string } | undefined
```

The constraint ensures every item has an `id`, but the full type (with `name` and `role`) is preserved in the return type.

### The `keyof` Constraint

A common pattern is constraining one parameter based on another:

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30, email: "alice@example.com" };
const name = getProperty(user, "name");  // string
const age = getProperty(user, "age");    // number
// getProperty(user, "phone");           // Error: 'phone' is not a key of user
```

The constraint `K extends keyof T` ensures the key is a valid property of the object, and the return type `T[K]` is the exact type of that property.

### Default Type Parameters

Type parameters can have defaults:

```typescript
interface PaginatedList<T, TMetadata = { total: number; page: number }> {
  items: T[];
  metadata: TMetadata;
}

// Using the default metadata type
const userList: PaginatedList<User> = {
  items: [{ id: "1", name: "Alice" }],
  metadata: { total: 100, page: 1 },
};

// Overriding the metadata type
const detailedList: PaginatedList<User, { total: number; page: number; hasMore: boolean }> = {
  items: [],
  metadata: { total: 0, page: 1, hasMore: false },
};
```

Default type parameters work like default function parameters — they provide a fallback when the type is not explicitly specified.

## Common Pitfalls
1. **Over-constraining** — Do not add constraints the function body does not need. If the function does not access `.length`, do not require it.
2. **Confusing `extends` in generics with class inheritance** — In a generic constraint, `T extends Shape` means "T must be assignable to Shape," not "T must be a subclass of Shape."

## Best Practices
1. **Use `keyof` constraints for property access** — The `getProperty` pattern ensures type-safe dynamic property access.
2. **Provide default type parameters for common cases** — This simplifies usage for the most common scenario while allowing customization.

## Summary
- `extends` constrains a generic to types matching a minimum shape.
- Named interfaces make complex constraints readable.
- `keyof` constraints enable type-safe property access patterns.
- Default type parameters simplify common usage while allowing overrides.

## Code Examples

**The keyof constraint ensures only valid property names are accepted, and the return type matches the property's type exactly**

```typescript
// keyof constraint for type-safe property access
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map(item => item[key]);
}

const products = [
  { name: "Keyboard", price: 79, inStock: true },
  { name: "Mouse", price: 29, inStock: false },
];

const names = pluck(products, "name");     // string[]
const prices = pluck(products, "price");   // number[]
// pluck(products, "color"); // Error: 'color' is not a key of the product type
```


## Resources

- [TypeScript Handbook: Generic Constraints](https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints) — Official guide to constraining generic type parameters

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*