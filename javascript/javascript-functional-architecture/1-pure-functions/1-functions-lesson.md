---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-pure-functions-lesson"
---

# Pure Functions

## Introduction

Pure functions are the foundation of functional programming. They're predictable, testable, and composable—making code easier to reason about.

## Key Concepts

**Deterministic**: Same input always produces same output.

**No Side Effects**: Doesn't modify external state.

**Referential Transparency**: Can replace call with its return value.

## Real World Context

React components are pure functions of props—same props, same JSX. Redux reducers must be pure for time-travel debugging to work. Testing pure functions is trivial: no mocking, no setup, just assert input→output. Every production codebase benefits from pushing logic into pure functions.

## Deep Dive

### Pure vs Impure

```javascript
// Pure - deterministic, no side effects
function add(a, b) {
  return a + b;
}

// Impure - uses external state
let total = 0;
function addToTotal(n) {
  total += n;  // Modifies external state
  return total;
}

// Impure - non-deterministic
function addRandom(n) {
  return n + Math.random();  // Different each time
}

// Impure - side effect
function addWithLog(a, b) {
  console.log(a, b);  // Side effect
  return a + b;
}
```

### Benefits of Pure Functions

```javascript
// 1. Easy to test
test('add', () => {
  expect(add(2, 3)).toBe(5);  // Always true
});

// 2. Memoizable
const memoAdd = memoize(add);  // Safe to cache

// 3. Parallelizable
const results = items.map(pure);  // No shared state

// 4. Composable
const process = compose(validate, transform, format);
```

## Common Pitfalls

1. **Hidden dependencies**: Relying on closure variables.
2. **Date/Random**: Using Date.now() or Math.random().
3. **I/O operations**: File, network, DOM access.

## Best Practices

1. **Push side effects to the edges** — Keep your core logic pure and handle I/O at the boundaries (API handlers, event listeners).
2. **Use `structuredClone()` for deep copies** — A built-in Web API (available since 2022) that guarantees no mutation of nested data.
3. **Treat Date.now() and Math.random() as dependencies** — Inject them as parameters so the function remains pure and testable.

## Summary

Pure functions are deterministic and side-effect free. They're testable, composable, and cacheable. Isolate impure code at the edges of your system.

## Code Examples

**Pure vs impure functions — pure functions have no side effects and always return the same output for the same input**

```javascript
// Pure — same input always gives same output
function add(a, b) {
  return a + b;
}

// Impure — depends on external state
let total = 0;
function addToTotal(n) {
  total += n; // Side effect: mutates external variable
  return total;
}

// Make it pure by returning new state
function addToTotal(currentTotal, n) {
  return currentTotal + n; // No mutation, returns new value
}
```


## Resources

- [MDN: Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions) — Functions guide

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*