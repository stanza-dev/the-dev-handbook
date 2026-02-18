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

## Summary

Recursion has base case and recursive case. Each call must move toward base case. Think of it as solving simpler version of same problem.

## Resources

- [MDN: Recursion](https://developer.mozilla.org/en-US/docs/Glossary/Recursion) — Recursion glossary

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*