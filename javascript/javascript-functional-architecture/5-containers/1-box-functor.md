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

## Real World Context

Arrays, Promises, and Observables are all functors—they all have a `map` (or `then`) method. React's component model is functor-like: props go in, JSX comes out, and you can map transforms over the output. Understanding functors makes you fluent with any container-based API.

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

## Common Pitfalls

1. **Forgetting to fold/extract** — Mapping transforms values inside the box but never extracts them. You need `fold` to get the final value out.
2. **Violating functor laws** — If `map(x => x)` doesn't return an identical container, your functor is broken. Always verify identity and composition laws.
3. **Overusing Box for simple values** — Wrapping a single value in Box just to call `.map()` adds ceremony without benefit. Use Box when you have a pipeline of transforms.

## Best Practices

1. **Use Box to linearize nested function calls** — `Box(x).map(f).map(g).fold(h)` is more readable than `h(g(f(x)))`.
2. **Follow functor laws** — `container.map(x => x)` must equal `container`, and `container.map(f).map(g)` must equal `container.map(x => g(f(x)))`.
3. **Think of Promise as a functor** — `.then(fn)` is essentially `.map(fn)`. Understanding this connection makes async code more intuitive.

## Summary

Functors have map. Map transforms without extracting. Box is the simplest functor. Arrays, Promises are functors. Laws ensure predictable behavior.

## Code Examples

**Box functor wraps a value and provides map for composable transformations — fold extracts the final result**

```javascript
const Box = x => ({
  map: f => Box(f(x)),
  fold: f => f(x),
  inspect: () => `Box(${x})`
});

// Linearize nested function calls
const result = Box('  Hello World  ')
  .map(s => s.trim())
  .map(s => s.toUpperCase())
  .map(s => s.split(' '))
  .fold(x => x);
// ['HELLO', 'WORLD']

// Functor laws
Box(5).map(x => x);           // Box(5) — identity law
Box(5).map(x => x + 1).map(x => x * 2); // Box(12)
Box(5).map(x => (x + 1) * 2);           // Box(12) — composition law
```


## Resources

- [MDN: Array.prototype.map() — Arrays as Functors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) — Official documentation: MDN: Array.prototype.map() — Arrays as Functors

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*