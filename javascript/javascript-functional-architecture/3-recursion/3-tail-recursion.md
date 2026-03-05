---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-tail-recursion"
---

# Tail Call Optimization

## Introduction

Tail Call Optimization (TCO) allows recursive functions to run in constant stack space. When the recursive call is the last operation, the engine can reuse the stack frame.

## Key Concepts

**Tail Position**: The last operation in a function before returning. If the recursive call is in tail position, the stack frame can be reused.

**Tail Call Optimization (TCO)**: The engine optimization that reuses stack frames for tail calls—constant stack space.

**Trampoline**: A manual TCO technique where recursive calls return thunks (no-arg functions) that a loop evaluates.

## Real World Context

Safari (JavaScriptCore) is the only major engine that implements TCO. V8 (Chrome, Node.js) does not. In practice, JavaScript developers use trampolines or convert to iteration for stack-safe recursion. Libraries like `trampoline` and `sanctuary` provide trampoline utilities.

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

## Common Pitfalls

1. **Assuming TCO works everywhere** — Only Safari supports it. Chrome/Firefox/Node.js will still overflow on deep recursion.
2. **Work after the recursive call** — `return n * factorial(n-1)` is NOT tail recursive because the multiply happens after the call returns.
3. **Trampoline overhead** — Trampolining adds function allocation overhead. Only use it when recursion depth could exceed the stack limit.

## Best Practices

1. **Convert to accumulator form first** — Move computation into an accumulator parameter so the recursive call is the last operation.
2. **Use trampoline for production safety** — Since TCO isn't universally supported, trampoline guarantees stack safety across all engines.
3. **Consider iteration when possible** — If the recursive structure is simple (linear), a `while` loop is more straightforward than a trampoline.

## Summary

TCO prevents stack overflow for tail recursive functions. Move computation into accumulator. Use trampoline for manual TCO when engine doesn't support it.

## Code Examples

**Tail recursion puts the recursive call in tail position; trampoline provides stack safety in engines without TCO**

```javascript
// Non-tail: work after recursive call (multiply)
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // NOT tail — multiply after call
}

// Tail recursive: call is the last operation
function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, n * acc); // Tail position
}

// Trampoline for stack-safe recursion in all engines
function trampoline(fn) {
  return function(...args) {
    let result = fn(...args);
    while (typeof result === 'function') result = result();
    return result;
  };
}

const safeFact = trampoline(function f(n, acc = 1) {
  if (n <= 1) return acc;
  return () => f(n - 1, n * acc); // Returns thunk, not recursive call
});
safeFact(100000); // Works without stack overflow
```


## Resources

- [MDN: Proper Tail Calls](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) — MDN documentation on arrow functions — commonly used in tail-recursive patterns

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*