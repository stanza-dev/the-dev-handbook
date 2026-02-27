---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-transducers"
---

# Transducers

## Introduction

Transducers are composable transformation functions that don't create intermediate collections. They're more efficient for large data sets.

## Deep Dive

### The Problem

```javascript
// Each operation creates intermediate array
[1, 2, 3, 4, 5]
  .map(x => x * 2)     // [2, 4, 6, 8, 10]
  .filter(x => x > 5)  // [6, 8, 10]
  .map(x => x + 1);    // [7, 9, 11]

// 3 iterations, 3 intermediate arrays!
```

### Transducer Concept

```javascript
// Transducers compose transformations, not data
const mapT = fn => reducer => (acc, x) => reducer(acc, fn(x));
const filterT = pred => reducer => (acc, x) => 
  pred(x) ? reducer(acc, x) : acc;

// Compose into single pass
const xform = compose(
  mapT(x => x * 2),
  filterT(x => x > 5),
  mapT(x => x + 1)
);

// Single iteration, no intermediates
const result = [1, 2, 3, 4, 5].reduce(
  xform((acc, x) => [...acc, x]),
  []
);
```

### With Libraries

```javascript
// Using ramda or transducers-js
import { transduce, map, filter, compose } from 'ramda';

const xform = compose(
  map(x => x * 2),
  filter(x => x > 5),
  map(x => x + 1)
);

transduce(xform, flip(append), [], [1, 2, 3, 4, 5]);
```

## Summary

Transducers compose transformations without intermediate arrays. Single pass over data. Use for performance-critical large data processing.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*