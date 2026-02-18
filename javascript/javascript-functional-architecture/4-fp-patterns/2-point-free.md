---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-point-free"
---

# Point-Free Style

## Introduction

Point-free (or tacit) programming defines functions without explicitly mentioning their arguments. It emphasizes composition over argument manipulation.

## Deep Dive

### Basic Examples

```javascript
// Pointed (explicit arguments)
const double = x => x * 2;
const doubleAll = arr => arr.map(x => double(x));

// Point-free
const doubleAll = arr => arr.map(double);

// Even more point-free with curry
const map = fn => arr => arr.map(fn);
const doubleAll = map(double);
```

### Composition

```javascript
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);

// Pointed
const processUser = user => {
  const name = user.name;
  const upper = name.toUpperCase();
  const trimmed = upper.trim();
  return trimmed;
};

// Point-free
const prop = key => obj => obj[key];
const toUpper = str => str.toUpperCase();
const trim = str => str.trim();

const processUser = compose(trim, toUpper, prop('name'));
```

### When to Use

```javascript
// Good: Simple, clear
const getNames = map(prop('name'));

// Too far: Hard to understand
const processData = compose(
  reduce(add, 0),
  filter(gt(10)),
  map(multiply(2)),
  pluck('value')
);  // What does this even do?
```

## Summary

Point-free removes argument mentions. Use for simple compositions. Don't sacrifice readability. Helper functions like prop, compose essential.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*