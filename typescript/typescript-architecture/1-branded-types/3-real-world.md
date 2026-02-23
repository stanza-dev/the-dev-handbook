---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-branded-real-world"
---

# Real-World Branded Type Patterns

## Introduction
Now that you understand the mechanics of branded types and smart constructors, this lesson covers the patterns used in production codebases: domain identifiers, monetary amounts, integration with schema validation libraries like Zod, and composing multiple brands.

## Key Concepts
- **Domain Identifier**: A branded string or number representing a database primary key or external API ID.
- **Monetary Type**: A branded number that encodes both the amount and currency, preventing arithmetic between incompatible currencies.
- **Schema-Driven Branding**: Using Zod's `.brand()` method to generate branded types directly from validation schemas.

## Real World Context
At companies like Stripe and Shopify, mixing up customer IDs, payment IDs, and order IDs is a real source of production bugs. Branded types in TypeScript eliminate this risk at zero runtime cost. Zod's `.brand()` bridges the gap between runtime validation and compile-time branding automatically.

## Deep Dive
### Domain Identifiers

Define one branded type per database entity:

```typescript
type Brand<B, T extends string> = B & { readonly __brand: T };

type UserId = Brand<string, "UserId">;
type ProductId = Brand<string, "ProductId">;
type OrderId = Brand<string, "OrderId">;

interface Order {
    id: OrderId;
    userId: UserId;
    productIds: ProductId[];
}
```

Every function that accepts an `OrderId` is now impossible to call with a `UserId`, even though both are strings at runtime.

### Monetary Amounts

Encode currency into the type to prevent cross-currency arithmetic:

```typescript
type USD = Brand<number, "USD">;
type EUR = Brand<number, "EUR">;

function addUsd(a: USD, b: USD): USD {
    return (a + b) as USD;
}

const price = 20 as USD;
const tax = 3 as USD;
const total = addUsd(price, tax); // OK: USD
// addUsd(price, 10 as EUR); // Error: EUR is not assignable to USD
```

### Zod Integration

Zod's `.brand()` method generates a branded type from a schema, combining runtime validation and compile-time branding:

```typescript
import { z } from "zod";

const EmailSchema = z.string().email().brand<"Email">();
type Email = z.infer<typeof EmailSchema>;

const result = EmailSchema.safeParse("alice@example.com");
if (result.success) {
    const email: Email = result.data; // Already branded
}
```

This eliminates the need for manual smart constructors when you already use Zod for validation.

### Composing Brands

You can combine multiple brands using intersection:

```typescript
type NonEmpty = Brand<string, "NonEmpty">;
type Trimmed = Brand<string, "Trimmed">;
type CleanString = NonEmpty & Trimmed;

function cleanString(input: string): CleanString | null {
    const trimmed = input.trim();
    if (trimmed.length === 0) return null;
    return trimmed as unknown as CleanString;
}
```

## Common Pitfalls
1. **Over-branding** — Not every string needs a brand. Use brands for values that cross module boundaries or represent domain identifiers. Local variables rarely benefit.
2. **Double-casting with composed brands** — When combining brands, you may need `as unknown as ComposedType` because TypeScript cannot directly narrow through multiple phantom properties.

## Best Practices
1. **Use Zod's `.brand()` at API boundaries** — Parse and brand incoming data in one step, then pass branded types throughout your application.
2. **Create a brands module** — Centralize all brand definitions and constructors in a single module for discoverability and consistency.

## Summary
- Domain identifiers (UserId, OrderId) prevent entity-ID mixups at compile time.
- Monetary branded types prevent cross-currency arithmetic errors.
- Zod's `.brand()` method unifies runtime validation with compile-time branding.
- Composed brands allow stacking multiple constraints on a single value.

## Code Examples

**Using Zod's .brand() to combine runtime UUID validation with compile-time UserId branding**

```typescript
import { z } from "zod";

// Zod schema with branding — validates AND brands in one step
const UserIdSchema = z.string().uuid().brand<"UserId">();
type UserId = z.infer<typeof UserIdSchema>;

// At the API boundary
function handleRequest(rawId: string) {
    const result = UserIdSchema.safeParse(rawId);
    if (!result.success) {
        throw new Error("Invalid user ID");
    }
    // result.data is already typed as UserId
    return fetchUser(result.data);
}

function fetchUser(id: UserId): void { /* ... */ }
```


## Resources

- [Zod Documentation: .brand()](https://zod.dev/?id=brand) — Official Zod docs on the .brand() method for creating branded types

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*