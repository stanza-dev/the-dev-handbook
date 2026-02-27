---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-recursive-patterns"
---

# Recursive Patterns

## Introduction

Recognizing common recursive patterns helps you apply them to new problems.

## Deep Dive

### Linear Recursion

```javascript
// Process first element, recurse on rest
function map(fn, arr) {
  if (arr.length === 0) return [];
  return [fn(arr[0]), ...map(fn, arr.slice(1))];
}

function filter(pred, arr) {
  if (arr.length === 0) return [];
  const [head, ...tail] = arr;
  return pred(head)
    ? [head, ...filter(pred, tail)]
    : filter(pred, tail);
}
```

### Tree Recursion

```javascript
// Fibonacci - two recursive calls
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}

// Tree traversal
function sumTree(node) {
  if (!node) return 0;
  return node.value + sumTree(node.left) + sumTree(node.right);
}
```

### Accumulator Pattern

```javascript
// Build result as you go
function reverse(arr, acc = []) {
  if (arr.length === 0) return acc;
  return reverse(arr.slice(1), [arr[0], ...acc]);
}

function factorial(n, acc = 1) {
  if (n <= 1) return acc;
  return factorial(n - 1, n * acc);
}
```

## Summary

Linear recursion processes one element at a time. Tree recursion makes multiple calls. Accumulator pattern builds result incrementally.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*