---
source_course: "javascript"
source_lesson: "javascript-loops"
---

# Loops: for, while, do-while

## Introduction

Loops let you execute code repeatedly—processing arrays, retrying operations, or waiting for conditions. JavaScript offers several loop types, each suited for different scenarios. Choosing the right loop makes your code clearer and more efficient.

## Key Concepts

**Iteration**: One execution of the loop body.

**Iterator Variable**: A variable that tracks the current position in the loop (often `i`).

**Infinite Loop**: A loop that never terminates—usually a bug that freezes your program.

## Real World Context

Loops process data: iterating through API responses, rendering lists of components, implementing pagination, polling for updates, and batch processing. Understanding loop performance matters when dealing with large datasets.

## Deep Dive

### Classic for Loop

```javascript
for (let i = 0; i < 5; i++) {
  console.log(i);  // 0, 1, 2, 3, 4
}

// Iterating arrays by index
const arr = ['a', 'b', 'c'];
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}
```

### while Loop

Use when the number of iterations is unknown:

```javascript
let attempts = 0;
while (attempts < 3) {
  const success = tryOperation();
  if (success) break;
  attempts++;
}
```

### do...while Loop

Always executes at least once:

```javascript
let input;
do {
  input = prompt('Enter a number greater than 10');
} while (Number(input) <= 10);
```

### Loop Control

```javascript
// break - exit the loop entirely
for (let i = 0; i < 10; i++) {
  if (i === 5) break;
  console.log(i);  // 0, 1, 2, 3, 4
}

// continue - skip to next iteration
for (let i = 0; i < 5; i++) {
  if (i === 2) continue;
  console.log(i);  // 0, 1, 3, 4
}

// Labels for nested loops
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) break outer;
    console.log(i, j);
  }
}
```

### Performance Tip

```javascript
// Cache array length for performance in hot loops
const arr = getLargeArray();
for (let i = 0, len = arr.length; i < len; i++) {
  process(arr[i]);
}
```

## Common Pitfalls

1. **Off-by-one errors**: Using `<=` instead of `<` or starting at 1 instead of 0.
2. **Infinite loops**: Forgetting to increment the counter or having an always-true condition.
3. **Modifying array while iterating**: Can skip elements or cause unexpected behavior.

## Best Practices

- **Use `for...of` for arrays**: When you don't need the index.
- **Use `for` when you need the index**: Or need to iterate in reverse.
- **Prefer array methods**: `map`, `filter`, `reduce` are often clearer than loops.
- **Avoid `while(true)`**: Unless you have a guaranteed break condition.

## Summary

The `for` loop is ideal when you know the iteration count. `while` loops check conditions before each iteration. `do...while` guarantees at least one execution. Use `break` to exit early and `continue` to skip iterations. For most array operations, prefer modern iterators or array methods.

## Resources

- [MDN: Loops and Iteration](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration) — Comprehensive guide to all loop types
- [MDN: for](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for) — for loop reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*