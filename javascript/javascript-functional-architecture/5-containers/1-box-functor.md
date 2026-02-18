---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-box-functor"
---

# The Box Pattern & Functors

## Introduction

A Functor is a container with a map method. It lets you transform the contained value without opening the container. This abstraction enables composable, safe data transformations.

## Key Concepts

**Functor**: Container implementing map.

**map**: Transform value inside without extracting.

**Identity Law**: container.map(x => x) equals container.

**Composition Law**: container.map(f).map(g) equals container.map(x => g(f(x))).

## Deep Dive

### The Box

```javascript
const Box = x => ({
  map: f => Box(f(x)),
  fold: f => f(x),
  inspect: () => `Box(${x})`
});

// Transform without extracting
Box(2)
  .map(x => x + 2)      // Box(4)
  .map(x => x * 3)      // Box(12)
  .fold(x => x);        // 12 (extract)
```

### Why Use It?

```javascript
// Without Box - imperative
const result = trim(toUpper(getValue(data)));

// With Box - composable pipeline
const result = Box(data)
  .map(getValue)
  .map(toUpper)
  .map(trim)
  .fold(x => x);
```

### Arrays are Functors

```javascript
// Array.map follows functor laws
[1, 2, 3].map(x => x);  // [1, 2, 3] (identity)
[1, 2, 3].map(x => x + 1).map(x => x * 2);  // [4, 6, 8]
[1, 2, 3].map(x => (x + 1) * 2);            // [4, 6, 8] (composition)
```

## Summary

Functors have map. Map transforms without extracting. Box is the simplest functor. Arrays, Promises are functors. Laws ensure predictable behavior.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*