---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-recursive-patterns"
---

# Recursive Patterns

## Introduction

Recognizing common recursive patterns helps you apply them to new problems.

## Key Concepts

**Linear Recursion**: One recursive call per invocation—processes a list one element at a time.

**Tree Recursion**: Multiple recursive calls per invocation—used for binary trees, divide-and-conquer.

**Accumulator Pattern**: Passing a running total through recursive calls to enable tail-call optimization.

## Real World Context

React component trees use tree recursion for rendering. Recursive descent parsers (used in Babel, TypeScript compiler) use linear and tree recursion to parse code. File system walkers recursively traverse directory trees. `Array.prototype.flat(Infinity)` internally uses recursion.

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

## Common Pitfalls

1. **Exponential time with tree recursion** — Naive Fibonacci (`fib(n-1) + fib(n-2)`) is O(2^n). Use memoization to make it O(n).
2. **Creating intermediate arrays with slice** — `arr.slice(1)` in each recursive call creates O(n) arrays, totaling O(n²) memory. Use an index parameter instead.
3. **Forgetting to accumulate** — Without an accumulator, values must be combined on the way "back up" the call stack, preventing tail-call optimization.

## Best Practices

1. **Use the accumulator pattern** — Pass results forward (`sum(arr, acc)`) instead of combining on return (`arr[0] + sum(rest)`).
2. **Use index parameters instead of slice** — `process(arr, i+1)` avoids creating new arrays at each level.
3. **Memoize tree-recursive functions** — Cache results to convert exponential to linear time complexity.

## Summary

Linear recursion processes one element at a time. Tree recursion makes multiple calls. Accumulator pattern builds result incrementally.

## Code Examples

**Accumulator builds the result as it recurses forward; tree recursion makes multiple calls per invocation**

```javascript
// Accumulator pattern — builds result forward
function reverse(arr, acc = []) {
  if (arr.length === 0) return acc;
  return reverse(arr.slice(1), [arr[0], ...acc]);
}

reverse([1, 2, 3]); // [3, 2, 1]

// Tree recursion — multiple recursive calls
function sumTree(node) {
  if (!node) return 0;
  return node.value + sumTree(node.left) + sumTree(node.right);
}

const tree = { value: 1, left: { value: 2, left: null, right: null }, right: { value: 3, left: null, right: null } };
sumTree(tree); // 6
```


## Resources

- [MDN: Functions — Recursion](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#recursion) — Official documentation: MDN: Functions — Recursion

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*