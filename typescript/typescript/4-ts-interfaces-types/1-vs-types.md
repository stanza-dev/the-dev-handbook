---
source_course: "typescript"
source_lesson: "typescript-interfaces-vs-types"
---

# Interfaces vs Type Aliases

## Introduction
TypeScript offers two ways to name object shapes: interfaces and type aliases. Both can describe objects, but they differ in capabilities and use cases. Understanding when to use each is one of the most common questions new TypeScript developers ask.

## Key Concepts
- **Interface**: A declaration that describes the shape of an object. Supports extension via `extends` and declaration merging.
- **Type Alias**: A name for any type, including primitives, unions, intersections, and objects. Uses the `type` keyword.
- **Declaration Merging**: A feature unique to interfaces where multiple declarations with the same name are automatically combined.
- **Intersection**: Combining types with `&` to create a new type that has all properties of both.

## Real World Context
In a large codebase, you will see both interfaces and types. Libraries like React use interfaces for component props (`React.FC<Props>`), while utility types often use type aliases. Knowing which to choose avoids inconsistency and leverages the right tool for each situation.

## Deep Dive

### Interfaces for Object Shapes

Interfaces are designed specifically for describing objects:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

interface AdminUser extends User {
  permissions: string[];
}

const admin: AdminUser = {
  id: "a1",
  name: "Alice",
  email: "alice@example.com",
  permissions: ["manage_users", "edit_content"],
};
```

The `extends` keyword creates a clean inheritance chain. TypeScript enforces that `AdminUser` includes all properties from `User` plus its own.

### Type Aliases for Everything Else

Type aliases can name any type, not just objects:

```typescript
type ID = string | number;
type Coordinate = [number, number];
type EventHandler = (event: MouseEvent) => void;

type ApiResponse<T> = {
  data: T;
  status: number;
  message: string;
};
```

For object shapes, type aliases use intersection (`&`) instead of `extends`:

```typescript
type Animal = { name: string; legs: number };
type Pet = Animal & { owner: string };

const dog: Pet = { name: "Rex", legs: 4, owner: "Bob" };
```

### Key Differences

The practical differences that matter:

```typescript
// 1. Declaration Merging — only interfaces
interface Window {
  customProperty: string;
}
// This merges with the global Window interface

// 2. Union types — only type aliases
type Status = "active" | "inactive" | "pending";
// Cannot do this with an interface

// 3. Tuple types — only type aliases
type Point = [number, number];

// 4. Computed properties — only type aliases
type Keys = "name" | "age";
type Person = { [K in Keys]: string };
```

Interfaces can do one thing that types cannot: declaration merging. Types can do several things interfaces cannot: unions, tuples, mapped types, and conditional types.

### When to Use Which

Use **interfaces** when you are defining the shape of an object that might be extended or merged (API contracts, component props, service definitions). Use **type aliases** when you need unions, tuples, mapped types, or any non-object type.

## Common Pitfalls
1. **Mixing `extends` and `&` inconsistently** — Pick one approach per codebase. If you use interfaces, extend with `extends`. If you use types, compose with `&`. Mixing them works but creates inconsistency.
2. **Not leveraging declaration merging** — When augmenting third-party types (like adding a property to Express's `Request`), declaration merging with interfaces is the correct approach.

## Best Practices
1. **Use interfaces for public API contracts** — They produce clearer error messages and support declaration merging for extensibility.
2. **Use type aliases for complex type operations** — Unions, intersections, mapped types, and conditional types require type aliases.

## Summary
- Interfaces describe object shapes and support `extends` and declaration merging.
- Type aliases name any type and support unions, tuples, and mapped types.
- Use interfaces for object contracts; use types for everything else.

## Code Examples

**Interfaces excel at describing object hierarchies, while type aliases handle unions, tuples, and other non-object types**

```typescript
// Interface: best for object shapes and API contracts
interface Product {
  id: string;
  name: string;
  price: number;
}

interface DigitalProduct extends Product {
  downloadUrl: string;
  fileSizeMb: number;
}

// Type alias: best for unions, tuples, and complex types
type PaymentMethod = "credit_card" | "paypal" | "bank_transfer";
type CartItem = { product: Product; quantity: number };
type CartTotal = [items: number, amount: number]; // labeled tuple
```


## Resources

- [TypeScript Handbook: Everyday Types (Interfaces vs Types)](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces) — Official comparison of interfaces and type aliases

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*