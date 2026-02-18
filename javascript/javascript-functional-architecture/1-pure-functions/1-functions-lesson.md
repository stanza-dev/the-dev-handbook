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

## Summary

Pure functions are deterministic and side-effect free. They're testable, composable, and cacheable. Isolate impure code at the edges of your system.

## Resources

- [MDN: Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions) — Functions guide

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*