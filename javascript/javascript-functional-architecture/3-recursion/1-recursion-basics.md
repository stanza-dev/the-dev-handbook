---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-recursion-basics"
---

# Recursion Fundamentals

## Introduction

Recursion solves problems by breaking them into smaller versions of the same problem. It's a powerful alternative to iteration and essential for tree structures.

## Key Concepts

**Base Case**: Condition that stops recursion.

**Recursive Case**: The self-call with smaller input.

**Call Stack**: Each call adds a frame; too deep causes overflow.

## Real World Context

Tree traversal (DOM, file systems, JSON), mathematical computations (factorial, Fibonacci), and divide-and-conquer algorithms all use recursion. React's reconciliation algorithm recursively diffs virtual DOM trees. `JSON.stringify` internally uses recursion to handle nested objects.

## Deep Dive

### Anatomy of Recursion

```javascript
function factorial(n) {
  // Base case - stops recursion
  if (n <= 1) return 1;
  
  // Recursive case - calls itself with smaller input
  return n * factorial(n - 1);
}

factorial(5);  // 5 * 4 * 3 * 2 * 1 = 120
```

### Tracing Execution

```javascript
factorial(4)
→ 4 * factorial(3)
→ 4 * (3 * factorial(2))
→ 4 * (3 * (2 * factorial(1)))
→ 4 * (3 * (2 * 1))
→ 4 * (3 * 2)
→ 4 * 6
→ 24
```

### Common Recursive Problems

```javascript
// Sum array
function sum(arr) {
  if (arr.length === 0) return 0;
  return arr[0] + sum(arr.slice(1));
}

// Count down
function countdown(n) {
  if (n <= 0) return;
  console.log(n);
  countdown(n - 1);
}
```

## Common Pitfalls

1. **Missing or incorrect base case** — Forgetting the base case causes infinite recursion and a Stack Overflow error.
2. **Not moving toward the base case** — Each recursive call must reduce the problem. `factorial(n)` calling `factorial(n)` loops forever.
3. **Stack overflow on large inputs** — JavaScript has a limited call stack (~10,000-25,000 frames). Use iteration or trampolining for deep recursion.

## Best Practices

1. **Write the base case first** — Start every recursive function by defining when it should stop.
2. **Verify progress toward base case** — Each recursive call's argument should be strictly closer to the base case than the caller's.
3. **Consider iteration for simple sequences** — Recursion shines for trees and nested structures; for flat sequences, a loop is often clearer and more efficient.

## Summary

Recursion has base case and recursive case. Each call must move toward base case. Think of it as solving simpler version of same problem.

## Code Examples

**Recursion fundamentals — every recursive function needs a base case (to stop) and a recursive case (that moves toward the base)**

```javascript
function factorial(n) {
  // Base case — stops the recursion
  if (n <= 1) return 1;
  // Recursive case — calls itself with smaller input
  return n * factorial(n - 1);
}

factorial(5); // 5 * 4 * 3 * 2 * 1 = 120

// Recursive array sum
function sum(arr) {
  if (arr.length === 0) return 0;    // Base case
  return arr[0] + sum(arr.slice(1)); // Recursive case
}

sum([1, 2, 3, 4]); // 10
```


## Resources

- [MDN: Recursion](https://developer.mozilla.org/en-US/docs/Glossary/Recursion) — Recursion glossary

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*