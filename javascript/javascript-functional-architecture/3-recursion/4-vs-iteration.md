---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-recursion-vs-iteration"
---

# Recursion vs Iteration

## Introduction

Most recursive solutions can be written iteratively and vice versa. Understanding when to use each is important.

## Key Concepts

**Iteration**: Repeating steps with loops (`for`, `while`). Uses constant stack space.

**Recursion**: A function calling itself. Uses stack space proportional to depth.

**Mutual Recursion**: Two or more functions calling each other—used in parser combinators and state machines.

## Real World Context

Most production JavaScript uses iteration for flat data and recursion for trees. React's fiber reconciler uses an iterative work loop for performance. Compilers use recursion for AST traversal. The choice is usually driven by data structure: flat → iterate, nested → recurse.

## Deep Dive

### Same Problem, Different Approaches

```javascript
// Iterative factorial
function factorialLoop(n) {
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}

// Recursive factorial
function factorialRec(n) {
  if (n <= 1) return 1;
  return n * factorialRec(n - 1);
}
```

### When to Use Recursion

```javascript
// Trees - naturally recursive
function traverse(node, fn) {
  if (!node) return;
  fn(node);
  traverse(node.left, fn);
  traverse(node.right, fn);
}

// Nested structures
function deepFlatten(arr) {
  return arr.reduce((flat, item) =>
    Array.isArray(item)
      ? [...flat, ...deepFlatten(item)]
      : [...flat, item]
  , []);
}
```

### When to Use Iteration

```javascript
// Simple sequences - iteration clearer
function sumArray(arr) {
  let sum = 0;
  for (const n of arr) sum += n;
  return sum;
}

// Performance critical - no call overhead
function findIndex(arr, pred) {
  for (let i = 0; i < arr.length; i++) {
    if (pred(arr[i])) return i;
  }
  return -1;
}
```

## Common Pitfalls

1. **Forcing recursion on flat data** — Recursively summing an array is elegant but slower and riskier than `reduce()`. Use the right tool.
2. **Converting tree operations to iteration poorly** — Iterative tree traversal requires an explicit stack/queue. Getting the order wrong produces subtle bugs.
3. **Ignoring stack limits** — JavaScript's call stack is limited. Processing a flat list of 100,000 items recursively will crash.

## Best Practices

1. **Match the approach to the data** — Trees and nested structures → recursion. Flat lists and sequences → iteration.
2. **Use `Array.prototype` methods** — `map`, `filter`, `reduce` are iterative HOFs that handle flat data cleanly.
3. **Benchmark when performance matters** — Iteration is generally faster due to no call stack overhead, but for small inputs the difference is negligible.

## Summary

Use recursion for trees and nested structures. Use iteration for simple sequences. Consider readability and performance.

## Code Examples

**Trees and nested structures suit recursion; flat sequences suit iteration — match the approach to the data shape**

```javascript
// Recursion: natural for tree structures
function traverse(node, fn) {
  if (!node) return;
  fn(node);
  traverse(node.left, fn);
  traverse(node.right, fn);
}

// Iteration: natural for flat sequences
function sumArray(arr) {
  let sum = 0;
  for (const n of arr) sum += n;
  return sum;
}

// Deep flatten — recursion handles nesting naturally
function deepFlatten(arr) {
  return arr.reduce((flat, item) =>
    Array.isArray(item)
      ? [...flat, ...deepFlatten(item)]
      : [...flat, item]
  , []);
}

deepFlatten([1, [2, [3, 4]], 5]); // [1, 2, 3, 4, 5]
```


## Resources

- [MDN: Array.prototype.flat()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat) — MDN reference on flat() for flattening nested arrays

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*