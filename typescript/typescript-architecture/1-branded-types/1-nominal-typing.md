---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-nominal-typing"
---

# Nominal vs Structural Typing

## Introduction
TypeScript uses structural typing: two types are compatible if their shapes match, regardless of their names. This is flexible but dangerous when you need to distinguish values that share the same primitive type. Branded types let you add compile-time identity to primitives so that a `UserId` can never be confused with an `OrderId`, even though both are strings underneath.

## Key Concepts
- **Structural Typing**: Types are compatible based on shape, not name. `{ x: number }` matches any object with a numeric `x` property.
- **Nominal Typing**: Types are compatible only when they share the same declared name. Languages like Java and C# use this model.
- **Branded Type**: A TypeScript pattern that simulates nominal typing by intersecting a base type with a phantom tag property.

## Real World Context
In a payment system, passing a raw `number` for both USD and EUR amounts is a recipe for currency-conversion bugs. Branded types make it a compile error to pass a `USD` amount where `EUR` is expected, catching the mistake before it reaches production.

## Deep Dive
TypeScript's structural type system treats these two types as fully interchangeable:

```typescript
type UserId = string;
type OrderId = string;

function getUser(id: UserId) { /* ... */ }
const orderId: OrderId = "order-123";
getUser(orderId); // No error — both are just string
```

This compiles without complaint because `UserId` and `OrderId` are both aliases for `string`. To fix this, we brand each type with a unique phantom property:

```typescript
type Brand<Base, Tag extends string> = Base & { readonly __brand: Tag };

type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;

function getUser(id: UserId) { /* ... */ }
const orderId = "order-123" as OrderId;
// getUser(orderId); // Error: 'OrderId' is not assignable to 'UserId'
```

The `__brand` property never exists at runtime — it is purely a compile-time tag. The intersection `string & { __brand: "UserId" }` creates a type that is structurally incompatible with `string & { __brand: "OrderId" }`, giving us nominal-like behavior.

A generic `Brand` utility makes this reusable across your entire codebase:

```typescript
type EUR = Brand<number, "EUR">;
type USD = Brand<number, "USD">;

function euroToUsd(amount: EUR): USD {
    return (amount * 1.1) as USD;
}

const price = 100 as EUR;
const converted = euroToUsd(price); // OK
// euroToUsd(100); // Error: number is not EUR
```

The `as` cast is required to create branded values because plain primitives lack the phantom property.

## Common Pitfalls
1. **Forgetting the cast** — You must use `as UserId` or a smart constructor to create branded values. A plain string will not type-check.
2. **Using `__brand` at runtime** — The brand property is a compile-time fiction. Never check `if (value.__brand)` in runtime code; it will always be `undefined`.

## Best Practices
1. **Use a single Brand utility** — Define `type Brand<B, T> = B & { readonly __brand: T }` once and reuse it everywhere for consistency.
2. **Prefer smart constructors over raw casts** — Wrap the `as` cast inside a validation function (covered in the next lesson) to prevent invalid branded values.

## Summary
- TypeScript is structurally typed; type aliases like `type UserId = string` offer no safety.
- Branded types use a phantom intersection property to simulate nominal typing.
- The brand exists only at compile time and has zero runtime cost.
- Use a generic `Brand<Base, Tag>` utility for consistency across the codebase.

## Code Examples

**A reusable Brand utility that creates compile-time nominal types from primitives**

```typescript
// Generic Brand utility
type Brand<Base, Tag extends string> = Base & { readonly __brand: Tag };

// Domain-specific branded types
type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;

// Creating branded values requires an explicit cast
const userId = "usr-42" as UserId;
const orderId = "ord-99" as OrderId;

// Type-safe function — only accepts UserId
function fetchUser(id: UserId): void { /* ... */ }

fetchUser(userId);   // OK
// fetchUser(orderId); // Error: OrderId is not assignable to UserId
```


## Resources

- [TypeScript Handbook: Type Compatibility](https://www.typescriptlang.org/docs/handbook/type-compatibility.html) — Official docs on structural typing and how TypeScript determines type compatibility
- [Branded Types in TypeScript](https://www.typescriptlang.org/play#example/nominal-typing) — TypeScript Playground example demonstrating nominal typing patterns

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*