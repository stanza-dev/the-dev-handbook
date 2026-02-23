---
source_course: "typescript"
source_lesson: "typescript-functions-objects"
---

# Functions and Object Types

## Introduction
Functions and objects are the core building blocks of any JavaScript application, and TypeScript provides precise ways to describe their shapes. Learning to type functions and objects correctly is essential for writing safe, maintainable code.

## Key Concepts
- **Parameter Types**: Annotations on function parameters that specify what callers must pass.
- **Return Type**: An annotation after the parameter list that declares what the function returns.
- **Object Type**: An inline type that describes the shape of an object using property names and their types.
- **Optional Property**: A property marked with `?` that may or may not be present on the object.

## Real World Context
Every REST API handler takes a request object and returns a response object. Typing these objects prevents sending malformed responses to clients. In React, component props are object types that define what data a component accepts.

## Deep Dive

### Function Types

Annotate parameters and optionally the return type:

```typescript
function calculateShipping(weight: number, destination: string): number {
  const ratePerKg = destination === "international" ? 15 : 5;
  return weight * ratePerKg;
}
```

The return type `: number` is optional here because TypeScript can infer it from the `return` statement. However, annotating return types on public functions is a good practice because it catches accidental changes.

### Arrow Functions

Arrow functions follow the same pattern:

```typescript
const formatName = (first: string, last: string): string => {
  return `${first} ${last}`;
};
```

For single-expression arrow functions, inference usually suffices:

```typescript
const double = (n: number) => n * 2; // return type inferred as number
```

### Object Types

Objects are typed by listing their properties and types:

```typescript
function printAddress(address: { street: string; city: string; zip: string }) {
  console.log(`${address.street}, ${address.city} ${address.zip}`);
}
```

Properties are separated by semicolons or commas. The type describes the minimum shape — the object can have additional properties.

### Optional Properties

Use `?` to mark properties that may not exist:

```typescript
function createUser(config: { name: string; email: string; bio?: string }) {
  // config.bio is string | undefined
  const biography = config.bio ?? "No bio provided";
  return { ...config, bio: biography };
}

createUser({ name: "Alice", email: "alice@example.com" }); // bio is optional
```

The `bio` property is `string | undefined` inside the function body, so you must handle the `undefined` case.

## Common Pitfalls
1. **Forgetting optional property checks** — Accessing an optional property without checking for `undefined` can cause runtime errors. Always use optional chaining (`?.`), nullish coalescing (`??`), or explicit checks.
2. **Overly broad object types** — Using `object` or `{}` as a type is almost as bad as `any`. Always describe the specific shape you expect.

## Best Practices
1. **Extract repeated object types into type aliases or interfaces** — If the same object shape appears in multiple places, define it once and reference it everywhere.
2. **Use readonly for function parameters** — Mark object parameters as `Readonly<>` when the function should not mutate them.

## Summary
- Annotate function parameters with types; return types are optional but recommended.
- Object types describe the exact shape of an object using property names and types.
- Optional properties use `?` and become `type | undefined` inside the function.

## Code Examples

**A function with a typed object parameter that includes an optional property with a union type default**

```typescript
// Object type with required and optional properties
function sendNotification(config: {
  userId: string;
  message: string;
  priority?: "low" | "high";
}): void {
  const level = config.priority ?? "low";
  console.log(`[${level}] To ${config.userId}: ${config.message}`);
}

sendNotification({ userId: "u123", message: "Your order shipped" });
sendNotification({ userId: "u456", message: "Payment failed", priority: "high" });
```


## Resources

- [TypeScript Handbook: More on Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html) — Official documentation on function types, signatures, and overloads
- [TypeScript Handbook: Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html) — Complete guide to object types, optional properties, and readonly

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*