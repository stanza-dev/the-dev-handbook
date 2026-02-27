---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-recursion-vs-iteration"
---

# Recursion vs Iteration

## Introduction

Most recursive solutions can be written iteratively and vice versa. Understanding when to use each is important.

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

## Summary

Use recursion for trees and nested structures. Use iteration for simple sequences. Consider readability and performance.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*