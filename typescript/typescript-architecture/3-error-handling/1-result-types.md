---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-result-types"
---

# The Result Type Pattern

## Introduction
JavaScript's `try/catch` mechanism has a critical flaw: thrown errors are invisible in function signatures. A function that can fail looks exactly like one that cannot. The Result pattern fixes this by encoding success and failure directly in the return type, forcing callers to handle both cases.

## Key Concepts
- **Result<T, E>**: A discriminated union representing either a successful value (`Ok<T>`) or an error (`Err<E>`).
- **Discriminant Property**: A shared property (like `success` or `ok`) that TypeScript uses to narrow the union.
- **Type-Safe Error Handling**: Callers must check the discriminant before accessing data, preventing unchecked error access.

## Real World Context
APIs that parse user input, query databases, or call external services can all fail. With `try/catch`, the error path is invisible — nothing in the type signature tells you the function can throw. With Result types, every failure mode is explicit and enforced by the compiler.

## Deep Dive
### Defining Result

```typescript
type Result<T, E = Error> =
    | { ok: true; value: T }
    | { ok: false; error: E };

function ok<T>(value: T): Result<T, never> {
    return { ok: true, value };
}

function err<E>(error: E): Result<never, E> {
    return { ok: false, error };
}
```

### Using Result

```typescript
function parseJson<T>(raw: string): Result<T, SyntaxError> {
    try {
        return ok(JSON.parse(raw));
    } catch (e) {
        return err(e as SyntaxError);
    }
}

const result = parseJson<{ name: string }>('{"name": "Alice"}');

if (result.ok) {
    console.log(result.value.name); // TypeScript knows value exists
} else {
    console.error(result.error.message); // TypeScript knows error exists
}
```

The discriminant `ok` narrows the type: inside the `if (result.ok)` branch, TypeScript guarantees `value` exists. In the `else` branch, it guarantees `error` exists.

### Typed Error Variants

Use string literal discriminants for multiple error types:

```typescript
type ParseError = { type: "PARSE_ERROR"; message: string };
type NotFoundError = { type: "NOT_FOUND"; id: string };
type AppError = ParseError | NotFoundError;

function findUser(id: string): Result<User, AppError> {
    // ...
}

const result = findUser("123");
if (!result.ok) {
    switch (result.error.type) {
        case "PARSE_ERROR":
            console.log(result.error.message);
            break;
        case "NOT_FOUND":
            console.log(`User ${result.error.id} not found`);
            break;
    }
}
```

## Common Pitfalls
1. **Mixing Result with try/catch** — If you wrap a function in Result but also throw inside it, callers get a false sense of safety. Contain all errors within Result.
2. **Forgetting to check the discriminant** — Accessing `result.value` without checking `result.ok` first is a type error. TypeScript enforces this, but be careful with type assertions.

## Best Practices
1. **Use Result for expected failures** — Network errors, validation failures, and missing data are expected. Use Result for these. Use exceptions only for truly unexpected bugs.
2. **Create domain-specific error types** — Instead of returning `Result<T, Error>`, define specific error types like `ValidationError` or `NotFoundError` for better downstream handling.

## Summary
- Result<T, E> encodes success and failure in the return type.
- The discriminant property (`ok`) enables TypeScript to narrow the union.
- Use typed error variants with a `type` discriminant for exhaustive switch handling.
- Reserve exceptions for unexpected bugs; use Result for expected failures.

## Code Examples

**A divide function that returns Result instead of throwing — the caller must handle the zero-division case**

```typescript
type Result<T, E = Error> =
    | { ok: true; value: T }
    | { ok: false; error: E };

function divide(a: number, b: number): Result<number, string> {
    if (b === 0) return { ok: false, error: "Division by zero" };
    return { ok: true, value: a / b };
}

const result = divide(10, 0);
if (result.ok) {
    console.log(result.value); // number — safe to use
} else {
    console.error(result.error); // "Division by zero"
}
```


## Resources

- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) — Official docs on discriminated unions and type narrowing

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*