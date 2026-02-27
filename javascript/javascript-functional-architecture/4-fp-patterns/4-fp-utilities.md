---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-fp-utilities"
---

# Essential FP Utilities

## Introduction

A toolkit of small, composable functions makes functional programming practical.

## Deep Dive

### Identity and Constant

```javascript
const identity = x => x;
const constant = x => () => x;

// Use identity for default transforms
const transform = fn || identity;

// Use constant for fixed values
map.defaultValue(constant(0));
```

### Property Access

```javascript
const prop = key => obj => obj[key];
const path = keys => obj => keys.reduce((o, k) => o?.[k], obj);

users.map(prop('name'));
orders.map(path(['customer', 'address', 'city']));
```

### Predicates

```javascript
const not = fn => (...args) => !fn(...args);
const both = (f, g) => (...args) => f(...args) && g(...args);
const either = (f, g) => (...args) => f(...args) || g(...args);

const isEven = n => n % 2 === 0;
const isPositive = n => n > 0;

const isPositiveEven = both(isEven, isPositive);
numbers.filter(isPositiveEven);
```

### Tap for Debugging

```javascript
const tap = fn => x => { fn(x); return x; };

data
  .map(transform)
  .filter(tap(console.log))  // Log but pass through
  .reduce(combine);
```

## Summary

Build a library of small utilities. identity, constant, prop, path are essential. Predicates compose with both/either/not. tap helps debugging.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*