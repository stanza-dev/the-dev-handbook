---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-side-effects"
---

# Managing Side Effects

## Introduction

Real programs need side effects—I/O, DOM updates, network calls. The key is isolating them from pure logic.

## Key Concepts

**Side Effect**: Any observable change outside the function's return value—mutations, I/O, logging, network calls.

**Effect Boundary**: The layer of your application where side effects are allowed (e.g., controllers, event handlers).

**Dependency Injection**: Passing effectful functions as parameters so the core logic remains pure.

## Real World Context

Every production app has side effects—database writes, API calls, DOM updates. The goal isn't to eliminate them but to isolate them. Frameworks like React enforce this: side effects go in `useEffect`, not in the render function. NestJS services separate pure business logic from effectful controllers.

## Deep Dive

### Identify Side Effects

```javascript
// Side effects:
// - Mutating arguments
// - Mutating external state
// - Console logging
// - DOM manipulation
// - Network requests
// - Reading/writing files
// - Getting current date/time
// - Random number generation
```

### Push Effects to the Edges

```javascript
// Bad: Mixed logic and side effects
function processUser(id) {
  const user = fetch(`/users/${id}`);  // Side effect
  const name = user.name.toUpperCase();  // Logic
  document.body.innerHTML = name;  // Side effect
}

// Good: Separate concerns
const formatName = name => name.toUpperCase();  // Pure

async function processUser(id) {
  const user = await fetchUser(id);  // Effect at edge
  const name = formatName(user.name);  // Pure logic
  render(name);  // Effect at edge
}
```

### Dependency Injection

```javascript
// Hard to test - hidden dependency
function getUser(id) {
  return fetch(`/users/${id}`);  // Hardcoded side effect
}

// Testable - inject dependency
function getUser(id, fetcher = fetch) {
  return fetcher(`/users/${id}`);
}

// Test with mock
getUser(1, mockFetch);
```

### Deterministic Cleanup with `using` (ES2025)

ES2025's `using` keyword enables automatic resource disposal when a block exits, making side-effect cleanup deterministic.

```javascript
{
  using connection = acquireDbConnection();
  // connection is automatically disposed when block exits
} // connection[Symbol.dispose]() called automatically
```

This replaces try/finally patterns and guarantees cleanup even if an exception is thrown.

## Common Pitfalls

1. **Hidden side effects in utility functions** — A `formatDate()` that calls `Date.now()` internally is impure. Pass the date as a parameter instead.
2. **Mutating function arguments** — `Array.sort()` mutates in place. Use `[...arr].sort()` or `Array.prototype.toSorted()` (ES2023) to avoid surprises.
3. **Global state in closures** — A function that reads or writes a module-level variable is impure, even if it looks self-contained.

## Best Practices

1. **Use the "functional core, imperative shell" pattern** — Pure business logic in the center, side effects at the edges.
2. **Make effects explicit in function signatures** — A function that fetches data should have `fetch` (or a client) as a parameter, not hidden inside.
3. **Test pure logic separately from effects** — Unit test the pure core, integration test the effectful shell.

## Summary

Isolate side effects at system edges. Keep core logic pure. Use dependency injection for testability.

## Code Examples

**Functional core, imperative shell — push side effects to the edges and keep core logic pure**

```javascript
// Impure: mixed logic and side effects
function processUser(id) {
  const user = fetchUser(id);          // Side effect
  const name = user.name.toUpperCase(); // Pure logic
  document.body.innerHTML = name;       // Side effect
}

// Better: separate pure logic from effects
const formatName = name => name.toUpperCase(); // Pure

async function processUser(id) {
  const user = await fetchUser(id);  // Effect at edge
  const name = formatName(user.name); // Pure core
  render(name);                       // Effect at edge
}
```


## Resources

- [MDN: Side Effects](https://developer.mozilla.org/en-US/docs/Glossary/Side_effect) — MDN reference on managing side effects in JavaScript

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*