---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-composition"
---

# Function Composition

## Introduction

Composition combines simple functions to build complex ones. It's the essence of functional programming—building programs by combining small, focused pieces.

## Key Concepts

**Compose**: Combines functions right-to-left: `compose(f, g)(x)` = `f(g(x))`.

**Pipe**: Combines functions left-to-right: `pipe(f, g)(x)` = `g(f(x))`. More readable for most developers.

**Point-Free**: Defining functions without explicitly mentioning arguments, enabled by composition.

## Real World Context

Unix pipes (`ls | grep | sort`) are composition. Express middleware chains compose request handlers. Data processing pipelines in ETL systems compose transformations. Ramda's `pipe()` and `compose()` are the most-used functions in functional JavaScript codebases.

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

## Common Pitfalls

1. **Compose direction confusion** — `compose(f, g)` runs g first, then f. `pipe(f, g)` runs f first. Pick one convention and stick with it.
2. **Error handling in pipelines** — If one function throws, the whole pipeline fails. Use Either/Result types for safe composition.
3. **Debugging composed functions** — A composed pipeline is opaque. Use `tap(console.log)` between stages to inspect intermediate values.

## Best Practices

1. **Prefer pipe over compose** — Left-to-right reads more naturally and matches the data flow direction.
2. **Keep composed functions small** — Each function in the pipeline should do one transformation. If a step is complex, extract and name it.
3. **Use tap for debugging** — `pipe(step1, tap(console.log), step2)` lets you inspect values without breaking the pipeline.

## Summary

Compose combines functions right-to-left. Pipe combines left-to-right. Point-free removes explicit data. Build complex from simple.

## Code Examples

**Pipe combines functions left-to-right — each step's output becomes the next step's input**

```javascript
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

// Build a data processing pipeline
const processUsers = pipe(
  users => users.filter(u => u.active),
  users => users.map(u => u.name),
  names => names.sort(),
  names => names.join(', ')
);

const users = [
  { name: 'Charlie', active: true },
  { name: 'Alice', active: true },
  { name: 'Bob', active: false }
];

processUsers(users); // 'Alice, Charlie'
```


## Resources

- [MDN: Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) — Official MDN documentation on reduce(), the foundation of many FP composition patterns

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*