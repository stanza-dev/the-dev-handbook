---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-hof"
---

# Higher-Order Functions

## Introduction

Higher-Order Functions (HOFs) take functions as arguments or return functions. They're the building blocks of functional composition.

## Key Concepts

**First-Class Functions**: Functions can be stored, passed, and returned.

**HOF**: A function that operates on other functions.

**Abstraction**: HOFs abstract common patterns.

## Real World Context

React hooks like `useCallback`, `useMemo`, and `useEffect` are HOFs—they take functions as arguments. Express middleware takes `(req, res, next) => {}` callbacks. Array methods `map`, `filter`, `reduce` are the most commonly used HOFs in any JavaScript codebase. ES2025 Iterator helpers add `.map()`, `.filter()` directly on iterators.

## Deep Dive

### Functions as Arguments

```javascript
// map, filter, reduce are HOFs
const numbers = [1, 2, 3, 4, 5];

numbers.map(x => x * 2);           // [2, 4, 6, 8, 10]
numbers.filter(x => x > 2);        // [3, 4, 5]
numbers.reduce((sum, x) => sum + x, 0);  // 15
```

### Functions Returning Functions

```javascript
function multiply(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiply(2);
const triple = multiply(3);

double(5);  // 10
triple(5);  // 15
```

### Creating HOFs

```javascript
// Repeat an action n times
function times(n) {
  return function(action) {
    for (let i = 0; i < n; i++) action(i);
  };
}

times(3)(i => console.log(i));  // 0, 1, 2

// Unless - inverse of filter
function unless(predicate) {
  return function(action) {
    return function(value) {
      if (!predicate(value)) action(value);
    };
  };
}
```

## Common Pitfalls

1. **Accidental closure over stale values** — A returned function captures variables by reference, not value. In loops, this often causes bugs where all closures share the same variable.
2. **Overusing HOFs for simple tasks** — `arr.map(x => x)` to clone an array is less clear than `[...arr]`. Use HOFs when they add clarity.
3. **Ignoring `this` binding** — Arrow functions capture `this` lexically, but `function` expressions don't. Returning a `function` from a HOF can lose `this` context.

## Best Practices

1. **Name returned functions** — Instead of returning anonymous functions, give them names for better stack traces: `return function handleClick() { ... }`.
2. **Keep HOFs focused** — A HOF should do one thing: add logging, add caching, add validation—not all three.
3. **Use ES2025 Iterator helpers for lazy HOFs** — `iterator.map(fn).filter(pred).take(n)` processes lazily without intermediate arrays.

## Summary

HOFs take or return functions. map/filter/reduce are common HOFs. Use them to abstract patterns and create reusable utilities.

## Code Examples

**Higher-Order Functions — map, filter, reduce take functions as arguments; multiply returns a new function**

```javascript
// HOFs take functions as arguments or return functions
const numbers = [1, 2, 3, 4, 5];

// Built-in HOFs
numbers.map(x => x * 2);          // [2, 4, 6, 8, 10]
numbers.filter(x => x > 2);       // [3, 4, 5]
numbers.reduce((sum, x) => sum + x, 0); // 15

// Creating a HOF that returns a function
function multiply(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiply(2);
const triple = multiply(3);
double(5); // 10
triple(5); // 15
```


## Resources

- [MDN: First-class functions](https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function) — First-class functions

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*