---
source_course: "typescript"
source_lesson: "typescript-type-aliases-unions-intersections"
---

# Type Aliases, Unions, and Intersections

## Introduction
Type aliases, unions, and intersections are three of TypeScript's most powerful composition tools. They let you build complex types from simple building blocks, like combining Lego bricks to create larger structures. Mastering these unlocks the full expressiveness of TypeScript's type system.

## Key Concepts
- **Type Alias**: A custom name for any type, created with the `type` keyword.
- **Union Type (`|`)**: A type that can be one of several options — "this OR that."
- **Intersection Type (`&`)**: A type that combines multiple types — "this AND that."
- **Discriminated Union**: A union where each member has a common property with a unique literal value, enabling safe narrowing.

## Real World Context
APIs return different shapes depending on success or failure. A union type like `SuccessResponse | ErrorResponse` models this naturally. Intersection types let you compose permissions, configurations, or feature flags without deep inheritance hierarchies.

## Deep Dive

### Type Aliases

A type alias gives a name to any type expression:

```typescript
type UserId = string;
type Timestamp = number;
type Callback = (error: Error | null, result: string) => void;

type UserProfile = {
  id: UserId;
  name: string;
  createdAt: Timestamp;
};
```

Type aliases are resolved at compile time — they are purely a convenience for readability and reuse. There is no runtime overhead.

### Union Types

Unions describe a value that can be one of several types:

```typescript
type StringOrNumber = string | number;
type ApiResult = SuccessResult | ErrorResult;
type Theme = "light" | "dark" | "system";
```

With a union, you can only access members that exist on all variants:

```typescript
function formatInput(input: string | number): string {
  // input.toUpperCase(); // Error: 'toUpperCase' does not exist on type 'number'
  return String(input); // Works for both types
}
```

To use type-specific methods, you need to narrow the type first (covered in a later section).

### Intersection Types

Intersections combine multiple types into one:

```typescript
type HasName = { name: string };
type HasEmail = { email: string };
type HasRole = { role: "admin" | "user" };

type AdminContact = HasName & HasEmail & HasRole;

const admin: AdminContact = {
  name: "Alice",
  email: "alice@example.com",
  role: "admin",
};
```

The resulting type must satisfy all three constituent types. This is the type-level equivalent of saying "this object must have a name AND an email AND a role."

### Composing Complex Types

Unions and intersections combine naturally:

```typescript
type SuccessResponse = {
  status: "success";
  data: unknown;
};

type ErrorResponse = {
  status: "error";
  message: string;
  code: number;
};

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse): void {
  if (response.status === "success") {
    console.log("Data:", response.data);
  } else {
    console.log(`Error ${response.code}: ${response.message}`);
  }
}
```

The `status` property acts as a discriminator, letting TypeScript narrow the type in each branch.

## Common Pitfalls
1. **Confusing union and intersection with OR and AND on values** — A union `A | B` means the value is A or B. An intersection `A & B` means the value is both A and B. For objects, intersection adds properties; it does not create an "either/or" choice.
2. **Impossible intersections** — `string & number` resolves to `never` because no value can be both. This is not an error, but it means no value satisfies the type.

## Best Practices
1. **Use unions for variant types** — When a value can be one of several shapes (API responses, event types, state machines), unions are the natural fit.
2. **Use intersections for composition** — When you want to combine capabilities (mixins, feature flags, augmenting types), intersections keep the code flat and composable.

## Summary
- Type aliases name any type for reuse and readability.
- Union types (`|`) describe values that can be one of several types.
- Intersection types (`&`) combine types, requiring all properties from each.
- Discriminated unions use a shared literal property for safe narrowing.

## Code Examples

**Intersection types compose reusable type fragments into complete entity types without class inheritance**

```typescript
// Building complex types through composition
type Timestamped = { createdAt: Date; updatedAt: Date };
type SoftDeletable = { deletedAt: Date | null };

type BaseEntity = Timestamped & SoftDeletable & { id: string };

type BlogPost = BaseEntity & {
  title: string;
  content: string;
  published: boolean;
};

// BlogPost has: id, createdAt, updatedAt, deletedAt, title, content, published
```


## Resources

- [TypeScript Handbook: Everyday Types (Union Types)](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types) — Official guide to union types and narrowing

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*