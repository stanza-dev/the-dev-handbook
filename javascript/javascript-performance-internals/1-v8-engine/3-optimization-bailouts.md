---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-optimization-bailouts"
---

# Optimization & Bailouts

## Introduction

Understanding what triggers optimization and what causes deoptimization helps you write code that stays fast.

## Deep Dive

### What Gets Optimized

```javascript
// Hot loops
for (let i = 0; i < 100000; i++) {
  processItem(items[i]);  // Gets optimized
}

// Frequently called functions
function hotFunction() {
  // Called many times → optimization candidate
}
```

### Deoptimization Triggers

```javascript
// 1. Type changes
function add(a, b) {
  return a + b;
}
add(1, 2);  // Numbers
add('a', 'b');  // Strings → deopt!

// 2. Hidden class changes
function process(obj) {
  return obj.x + obj.y;
}
process({ x: 1, y: 2 });  // Shape A
process({ y: 2, x: 1 });  // Shape B → polymorphic

// 3. Out-of-bounds array access
const arr = [1, 2, 3];
arr[100];  // Out of bounds
arr[-1];   // Negative index

// 4. Sparse arrays
const sparse = [];
sparse[0] = 1;
sparse[10000] = 2;  // Hole! → slower
```

### Checking Optimization Status

```javascript
// In Node.js with --allow-natives-syntax
function test(x) {
  return x * x;
}

// Warm up
for (let i = 0; i < 100000; i++) test(i);

// Check status
%OptimizationStatus(test);
// Returns optimization state flags
```

## Best Practices

- **Avoid arguments object**: Use rest parameters.
- **Don't reassign function parameters**: Causes deopt.
- **Use typed arrays for numeric work**: Predictable types.
- **Avoid sparse arrays**: Use Map for sparse data.

## Summary

Optimization happens after code runs many times. Deoptimization triggered by type changes, shape changes, and special cases. Keep types consistent in hot paths.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*