---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-branded-validation"
---

# Smart Constructors & Runtime Validation

## Introduction
Branding a type with `as` is easy, but it trusts the caller to provide a valid value. Smart constructors combine branded types with runtime validation so that if a value passes the constructor, it is guaranteed to be both valid and correctly branded.

## Key Concepts
- **Smart Constructor**: A function that validates input at runtime and returns a branded type on success or an error on failure.
- **Opaque Type**: A branded type whose creation is restricted to a single module, preventing arbitrary casting elsewhere.
- **Validation Function**: A runtime check (regex, range, business rule) applied before branding.

## Real World Context
An `Email` branded type is useless if you can write `"not-an-email" as Email`. Smart constructors enforce that every `Email` value has actually passed a regex check, eliminating an entire category of invalid-data bugs at the boundary where user input enters your system.

## Deep Dive
A smart constructor returns a `Result`-like type so callers must handle the failure case:

```typescript
type Brand<B, T extends string> = B & { readonly __brand: T };
type Email = Brand<string, "Email">;

function createEmail(input: string): Email {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!regex.test(input)) {
        throw new Error(`Invalid email: ${input}`);
    }
    return input as Email;
}

const valid = createEmail("alice@example.com"); // Email
// createEmail("bad");  // throws at runtime
```

For a safer API without exceptions, return a discriminated union:

```typescript
type Result<T> = { ok: true; value: T } | { ok: false; error: string };

function parseEmail(input: string): Result<Email> {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!regex.test(input)) {
        return { ok: false, error: `Invalid email: ${input}` };
    }
    return { ok: true, value: input as Email };
}
```

To make the brand truly opaque, export only the type and the constructor — never export a raw `as` cast:

```typescript
// email.ts — the ONLY place that casts to Email
export type { Email };
export { parseEmail };
// Consumers cannot write `as Email` because the brand tag is not exported
```

You can also use assertion functions for an ergonomic throwing API:

```typescript
function assertEmail(input: string): asserts input is Email {
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(input)) {
        throw new Error(`Invalid email: ${input}`);
    }
}
```

## Common Pitfalls
1. **Exposing the brand tag** — If consumers can import the brand tag, they can bypass the constructor with `as Email`. Keep the tag private to the module.
2. **Skipping validation in tests** — Even in tests, use the smart constructor. Using `as Email` in tests means your tests do not exercise the validation logic.

## Best Practices
1. **One constructor per branded type** — Centralize creation to make validation changes easy and auditable.
2. **Return Result instead of throwing** — Exceptions are invisible in TypeScript signatures. Returning a union makes the failure path explicit.

## Summary
- Smart constructors wrap branded type creation with runtime validation.
- Returning a Result union is safer than throwing exceptions.
- Keep the brand tag private to prevent bypass via raw casting.
- Assertion functions (`asserts input is T`) offer a concise throwing alternative.

## Code Examples

**A smart constructor that validates the input is a positive integer before branding it**

```typescript
type Brand<B, T extends string> = B & { readonly __brand: T };
type PositiveInt = Brand<number, "PositiveInt">;

// Smart constructor with Result return
function toPositiveInt(n: number): PositiveInt {
    if (!Number.isInteger(n) || n <= 0) {
        throw new Error(`Expected positive integer, got ${n}`);
    }
    return n as PositiveInt;
}

const quantity = toPositiveInt(5);  // PositiveInt
// toPositiveInt(-1);  // throws Error
// toPositiveInt(3.5); // throws Error
```


## Resources

- [TypeScript Handbook: Assertion Functions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions) — Official docs on assertion functions that narrow types via runtime checks

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*