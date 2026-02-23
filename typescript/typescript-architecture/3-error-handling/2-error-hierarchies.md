---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-error-hierarchies"
---

# Typed Error Hierarchies

## Introduction
When a function can fail in multiple ways, you need typed error hierarchies — discriminated unions where each variant carries error-specific data. Combined with exhaustive switch statements and the `never` type, TypeScript ensures you handle every possible error.

## Key Concepts
- **Error Hierarchy**: A union of error types, each with a unique `type` discriminant.
- **Exhaustive Switch**: A switch statement that handles every variant of a union. TypeScript uses the `never` type to verify completeness.
- **assertNever**: A helper function that produces a compile error if a switch is not exhaustive.

## Real World Context
A user registration flow can fail with invalid email, duplicate username, weak password, or rate limiting. Each error requires different handling (show field error, suggest alternatives, redirect to login). A typed error hierarchy ensures no failure mode is silently ignored.

## Deep Dive
### Defining an Error Hierarchy

```typescript
type ValidationError = {
    type: "VALIDATION";
    field: string;
    message: string;
};

type DuplicateError = {
    type: "DUPLICATE";
    entity: string;
    conflictingId: string;
};

type NetworkError = {
    type: "NETWORK";
    statusCode: number;
    retryable: boolean;
};

type AppError = ValidationError | DuplicateError | NetworkError;
```

### Exhaustive Handling with never

```typescript
function assertNever(value: never): never {
    throw new Error(`Unhandled case: ${JSON.stringify(value)}`);
}

function handleError(error: AppError): string {
    switch (error.type) {
        case "VALIDATION":
            return `Field '${error.field}': ${error.message}`;
        case "DUPLICATE":
            return `${error.entity} already exists (${error.conflictingId})`;
        case "NETWORK":
            return error.retryable
                ? `Network error (${error.statusCode}), retrying...`
                : `Fatal network error: ${error.statusCode}`;
        default:
            return assertNever(error);
    }
}
```

If you add a new error variant to `AppError` but forget to add a case, TypeScript will error on the `assertNever(error)` call because `error` will not be `never` — it will still have the unhandled variant's type.

### Combining with Result

```typescript
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

function registerUser(data: RegistrationData): Result<User, AppError> {
    if (!isValidEmail(data.email)) {
        return { ok: false, error: { type: "VALIDATION", field: "email", message: "Invalid format" } };
    }
    // ... more checks
    return { ok: true, value: newUser };
}
```

## Common Pitfalls
1. **Forgetting the default case** — Without `assertNever` in the default branch, adding a new error variant will not produce a compile error, silently dropping the new case.
2. **Using string instead of literal types** — `type: string` prevents narrowing. Always use string literals like `type: "VALIDATION"`.

## Best Practices
1. **Always include assertNever** — Every exhaustive switch should have a `default: return assertNever(value)` guard.
2. **Group related errors** — Create sub-unions for related errors (e.g., `type AuthError = InvalidCredentials | Expired | Locked`) and compose them into a top-level `AppError`.

## Summary
- Error hierarchies use discriminated unions with a `type` string literal.
- `assertNever` in the default branch ensures exhaustive handling at compile time.
- Adding a new error variant without a handler becomes a compile error.
- Compose sub-unions for modular error hierarchies.

## Code Examples

**assertNever ensures every Shape variant is handled — adding a new shape without a case causes a compile error**

```typescript
function assertNever(value: never): never {
    throw new Error(`Unhandled: ${JSON.stringify(value)}`);
}

type Shape =
    | { kind: "circle"; radius: number }
    | { kind: "rect"; width: number; height: number };

function area(shape: Shape): number {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "rect":
            return shape.width * shape.height;
        default:
            return assertNever(shape); // Compile error if a variant is missing
    }
}
```


## Resources

- [TypeScript Handbook: Discriminated Unions](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions) — Official docs on discriminated unions and exhaustive checking

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*