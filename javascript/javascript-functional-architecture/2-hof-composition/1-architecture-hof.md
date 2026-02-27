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

## Summary

HOFs take or return functions. map/filter/reduce are common HOFs. Use them to abstract patterns and create reusable utilities.

## Resources

- [MDN: First-class functions](https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function) — First-class functions

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*