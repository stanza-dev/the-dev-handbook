---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-effect-error-handling"
---

# Chaining Results with Map and FlatMap

## Introduction
Checking `if (result.ok)` at every step creates deeply nested code. The map/flatMap pattern lets you chain operations on Result values — transforming successes while automatically propagating errors — producing flat, readable pipelines.

## Key Concepts
- **map**: Transforms the success value inside a Result without unwrapping it. Errors pass through unchanged.
- **flatMap (chain)**: Like map, but the transformation itself returns a Result, avoiding nested `Result<Result<T>>`.
- **mapError**: Transforms the error value, useful for converting low-level errors into domain errors.

## Real World Context
A request handler that parses a body, validates fields, looks up a user, and saves to a database involves four operations that can each fail. Without chaining, you write four nested `if` blocks. With flatMap, it becomes a flat pipeline.

## Deep Dive
### Implementing Map and FlatMap

```typescript
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

function map<T, U, E>(result: Result<T, E>, fn: (value: T) => U): Result<U, E> {
    return result.ok ? { ok: true, value: fn(result.value) } : result;
}

function flatMap<T, U, E>(result: Result<T, E>, fn: (value: T) => Result<U, E>): Result<U, E> {
    return result.ok ? fn(result.value) : result;
}

function mapError<T, E, F>(result: Result<T, E>, fn: (error: E) => F): Result<T, F> {
    return result.ok ? result : { ok: false, error: fn(result.error) };
}
```

### Chaining Operations

```typescript
function parseId(raw: string): Result<number, string> {
    const n = parseInt(raw, 10);
    return isNaN(n) ? { ok: false, error: "Invalid ID" } : { ok: true, value: n };
}

function findUser(id: number): Result<User, string> {
    const user = db.get(id);
    return user ? { ok: true, value: user } : { ok: false, error: "User not found" };
}

function getUserEmail(rawId: string): Result<string, string> {
    return map(
        flatMap(parseId(rawId), findUser),
        (user) => user.email
    );
}
```

The chain reads: parse the ID, then find the user, then extract the email. If any step fails, the error propagates to the final result without additional `if` checks.

### Pipe-style chaining

For better readability, combine with a pipe function:

```typescript
const result = pipe(
    parseId("42"),
    (r) => flatMap(r, findUser),
    (r) => map(r, (user) => user.email)
);
```

## Common Pitfalls
1. **Using map when you need flatMap** — If your transformation returns a Result, use flatMap. Using map would produce `Result<Result<T>>`, which is unusable.
2. **Ignoring the error type** — When chaining operations with different error types, you need to unify them (e.g., with `mapError`) or use a common error union.

## Best Practices
1. **Keep individual steps small** — Each function in the chain should do one thing: parse, validate, lookup, or transform.
2. **Use a library for production code** — Libraries like `neverthrow`, `fp-ts`, or `Effect` provide battle-tested Result/Either implementations with rich chaining APIs.

## Summary
- `map` transforms success values; errors pass through unchanged.
- `flatMap` chains operations that themselves return Results, keeping the pipeline flat.
- `mapError` transforms error values for domain-level error conversion.
- For production code, consider `neverthrow` or `Effect` instead of hand-rolling.

## Code Examples

**Chaining Result operations with flatMap — errors short-circuit automatically without nested if/else**

```typescript
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

const ok = <T>(value: T): Result<T, never> => ({ ok: true, value });
const err = <E>(error: E): Result<never, E> => ({ ok: false, error });

const flatMap = <T, U, E>(r: Result<T, E>, fn: (v: T) => Result<U, E>): Result<U, E> =>
    r.ok ? fn(r.value) : r;

const map = <T, U, E>(r: Result<T, E>, fn: (v: T) => U): Result<U, E> =>
    r.ok ? ok(fn(r.value)) : r;

// Chain: parse → validate → transform
const result = flatMap(
    flatMap(ok("42"), s => {
        const n = Number(s);
        return isNaN(n) ? err("NaN") : ok(n);
    }),
    n => n > 0 ? ok(n * 2) : err("Not positive")
);
// result: { ok: true, value: 84 }
```


## Resources

- [neverthrow — Type-Safe Errors for TypeScript](https://github.com/supermacro/neverthrow) — A popular library providing Result type with map, flatMap, and match methods

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*