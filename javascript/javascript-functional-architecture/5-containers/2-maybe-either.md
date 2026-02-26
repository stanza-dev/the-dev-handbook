---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-maybe-either"
---

# Maybe & Either

## Introduction

Maybe and Either are functors that handle absence and errors safely. They eliminate null checks and try/catch boilerplate.

## Key Concepts

**Maybe (Option)**: A container that is either `Just(value)` or `Nothing`. Represents the presence or absence of a value.

**Either (Result)**: A container that is either `Right(value)` (success) or `Left(error)` (failure).

**Short-Circuiting**: `Nothing.map(fn)` and `Left.map(fn)` skip the function entirely, propagating the empty/error state.

## Real World Context

Optional chaining (`?.`) is JavaScript's built-in Maybe. `Promise.try()` (ES2025) wraps synchronous functions that might throw into Promise-based Either-like error handling. Libraries like fp-ts and neverthrow bring typed Maybe/Either to TypeScript codebases. Rust's `Option` and `Result` types are the same concept.

## Deep Dive

### Maybe (Option)

```javascript
const Just = x => ({
  map: f => Just(f(x)),
  fold: (onNothing, onJust) => onJust(x),
  isNothing: false
});

const Nothing = () => ({
  map: f => Nothing(),
  fold: (onNothing, onJust) => onNothing(),
  isNothing: true
});

const Maybe = x => x == null ? Nothing() : Just(x);

// Safe property access
Maybe(user)
  .map(u => u.address)
  .map(a => a.street)
  .fold(
    () => 'No street',
    street => street
  );
```

### Either (Result)

```javascript
const Right = x => ({
  map: f => Right(f(x)),
  fold: (onLeft, onRight) => onRight(x),
  isLeft: false
});

const Left = x => ({
  map: f => Left(x),  // Doesn't transform!
  fold: (onLeft, onRight) => onLeft(x),
  isLeft: true
});

// Error handling
const parseJSON = str => {
  try {
    return Right(JSON.parse(str));
  } catch (e) {
    return Left(e.message);
  }
};

parseJSON('{"a":1}')
  .map(obj => obj.a)
  .fold(
    err => console.error(err),
    val => console.log(val)
  );
```

## Common Pitfalls

1. **Nesting Maybes** — `Maybe(user).map(u => Maybe(u.address))` creates `Maybe(Maybe(address))`. Use `flatMap`/`chain` instead.
2. **Ignoring the Left case** — Always handle both branches with `fold(onLeft, onRight)`. Ignoring errors defeats the purpose.
3. **Converting everything to Maybe** — Not every null check needs Maybe. Use it for chains of potentially-absent values, not one-off checks.

## Best Practices

1. **Use `fold` to extract values** — Always provide handlers for both cases: `maybe.fold(() => default, x => x)`.
2. **Use `flatMap` for chaining** — When your mapping function returns a Maybe/Either, use `flatMap` to avoid nesting.
3. **Use `Promise.try()` (ES2025) for sync-to-async** — `Promise.try(() => JSON.parse(str))` catches sync errors and wraps them in a rejected Promise, similar to Either's Left.

## Summary

Maybe handles null/undefined safely. Either handles success/error. Left short-circuits (doesn't map). fold extracts with handling for both cases.

## Code Examples

**Maybe wraps nullable values — Nothing short-circuits map/flatMap, preventing null reference errors**

```javascript
const Just = x => ({
  map: f => Just(f(x)),
  flatMap: f => f(x),
  fold: (onNothing, onJust) => onJust(x),
  isNothing: false
});

const Nothing = () => ({
  map: f => Nothing(),
  flatMap: f => Nothing(),
  fold: (onNothing, onJust) => onNothing(),
  isNothing: true
});

const Maybe = x => (x == null ? Nothing() : Just(x));

// Safe nested property access
const street = Maybe(user)
  .flatMap(u => Maybe(u.address))
  .flatMap(a => Maybe(a.street))
  .fold(
    () => 'No street found',
    street => street
  );
```


## Resources

- [MDN: Optional Chaining (?.)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining) — MDN reference on optional chaining for safe property access

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*