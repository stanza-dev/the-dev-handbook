---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-composition"
---

# Function Composition

## Introduction

Composition combines simple functions to build complex ones. It's the essence of functional programming—building programs by combining small, focused pieces.

## Deep Dive

### Basic Compose

```javascript
// f(g(x)) - apply g, then f
const compose = (f, g) => x => f(g(x));

const addOne = x => x + 1;
const double = x => x * 2;

const doubleThenAddOne = compose(addOne, double);
doubleThenAddOne(5);  // 11 (5 * 2 + 1)

const addOneThenDouble = compose(double, addOne);
addOneThenDouble(5);  // 12 ((5 + 1) * 2)
```

### Compose Many Functions

```javascript
const compose = (...fns) => x =>
  fns.reduceRight((acc, fn) => fn(acc), x);

// Right-to-left: trim → lowercase → split
const processString = compose(
  str => str.split(' '),
  str => str.toLowerCase(),
  str => str.trim()
);

processString('  Hello World  ');
// ['hello', 'world']
```

### Pipe (Left-to-Right)

```javascript
const pipe = (...fns) => x =>
  fns.reduce((acc, fn) => fn(acc), x);

// Left-to-right: more readable
const processString = pipe(
  str => str.trim(),
  str => str.toLowerCase(),
  str => str.split(' ')
);
```

### Point-Free Style

```javascript
// With points (explicit data)
const getLengths = arr => arr.map(s => s.length);

// Point-free (no explicit data)
const length = s => s.length;
const getLengths = map(length);
```

## Summary

Compose combines functions right-to-left. Pipe combines left-to-right. Point-free removes explicit data. Build complex from simple.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*