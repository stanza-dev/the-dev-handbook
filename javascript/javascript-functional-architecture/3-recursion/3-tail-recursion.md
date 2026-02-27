---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-tail-recursion"
---

# Tail Call Optimization

## Introduction

Tail Call Optimization (TCO) allows recursive functions to run in constant stack space. When the recursive call is the last operation, the engine can reuse the stack frame.

## Deep Dive

### Non-Tail vs Tail Recursive

```javascript
// Non-tail - work after recursive call
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);  // Multiply AFTER return
}

// Tail recursive - call is last operation
function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, n * acc);  // Nothing after call
}
```

### Converting to Tail Form

```javascript
// Non-tail sum
function sum(arr) {
  if (arr.length === 0) return 0;
  return arr[0] + sum(arr.slice(1));
}

// Tail recursive sum
function sumTail(arr, acc = 0) {
  if (arr.length === 0) return acc;
  return sumTail(arr.slice(1), acc + arr[0]);
}
```

### Trampoline (Manual TCO)

```javascript
function trampoline(fn) {
  return function(...args) {
    let result = fn(...args);
    while (typeof result === 'function') {
      result = result();
    }
    return result;
  };
}

function factorialT(n, acc = 1) {
  if (n <= 1) return acc;
  return () => factorialT(n - 1, n * acc);
}

const factorial = trampoline(factorialT);
factorial(100000);  // Works without overflow
```

## Summary

TCO prevents stack overflow for tail recursive functions. Move computation into accumulator. Use trampoline for manual TCO when engine doesn't support it.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*