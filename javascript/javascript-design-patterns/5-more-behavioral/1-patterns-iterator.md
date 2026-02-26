---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-iterator"
---

# The Iterator Pattern

## Introduction

The Iterator pattern provides a way to access elements of a collection sequentially without exposing its underlying representation. JavaScript has this built in with the Iterator protocol.

## Key Concepts

**Iterable**: An object with a `[Symbol.iterator]()` method.

**Iterator**: An object with a `next()` method that returns `{ value, done }`.

**Iterator Protocol**: The contract that enables `for...of`, spread, and destructuring.

**ES2025 Iterator Helpers**: New built-in methods like `.map()`, `.filter()`, `.take()`, `.drop()` directly on iterators—no conversion to Array needed.

## Real World Context

DOM NodeLists, Map/Set, generators, and streams all implement the Iterator protocol. ES2025 Iterator helpers let you chain `.map()`, `.filter()`, `.take()` directly on any iterator without converting to an Array first—huge for performance with large or infinite sequences.

## Deep Dive

### Custom Iterator

Implementing `[Symbol.iterator]()` makes any object work with `for...of`, spread, and destructuring. This `Range` class yields integers from `start` to `end`:

```javascript
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  
  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    
    return {
      next() {
        if (current <= end) {
          return { value: current++, done: false };
        }
        return { done: true };
      }
    };
  }
}

for (const n of new Range(1, 5)) {
  console.log(n); // 1, 2, 3, 4, 5
}

[...new Range(1, 3)]; // [1, 2, 3]
```

The iterator returns `{ value, done }` objects. When `current` exceeds `end`, it returns `{ done: true }`, signaling that iteration is complete.

### Generator-Based Iterator

Generators simplify iterator creation by using `yield` to produce values and `yield*` to delegate to child iterators. No manual `{ value, done }` management is needed:

```javascript
class Tree {
  constructor(value, children = []) {
    this.value = value;
    this.children = children;
  }
  
  *[Symbol.iterator]() {
    yield this.value;
    for (const child of this.children) {
      yield* child;
    }
  }
}

const tree = new Tree(1, [
  new Tree(2, [new Tree(4)]),
  new Tree(3)
]);
[...tree]; // [1, 2, 4, 3]
```

The `yield*` keyword delegates iteration to each child's `[Symbol.iterator]()`, producing a depth-first traversal of the entire tree with minimal code.

## Common Pitfalls

1. **Consuming an iterator twice** — Iterators are stateful and single-use. Calling `[...iter]` exhausts it; a second spread yields nothing.
2. **Forgetting `done: true`** — If `next()` never returns `{ done: true }`, `for...of` loops run forever.
3. **Confusing iterable with iterator** — An iterable has `[Symbol.iterator]()`, an iterator has `next()`. They can be the same object, but they're distinct concepts.

## Best Practices

1. **Return the iterator from `[Symbol.iterator]()`** — This makes your object work with `for...of`, spread, and destructuring.
2. **Use generators for complex iterators** — `function*` simplifies iterator creation and handles `done` automatically.
3. **Use ES2025 Iterator helpers for lazy processing** — `.map()`, `.filter()`, `.take()` on iterators avoid creating intermediate arrays.

## Summary

Iterators provide sequential access to collections. JavaScript's Symbol.iterator enables for...of and spread. Generators make creating iterators simple.

## Code Examples

**Custom iterable Range class with ES2025 Iterator helpers — .filter(), .take(), .toArray() work directly on iterators**

```javascript
class Range {
  constructor(start, end) { this.start = start; this.end = end; }

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    return {
      next() {
        return current <= end
          ? { value: current++, done: false }
          : { done: true };
      }
    };
  }
}

// Works with for...of, spread, destructuring
for (const n of new Range(1, 5)) console.log(n); // 1,2,3,4,5

// ES2025 Iterator helpers — chain directly on iterators
const evens = new Range(1, 10)[Symbol.iterator]()
  .filter(n => n % 2 === 0)
  .take(3)
  .toArray(); // [2, 4, 6]
```


## Resources

- [MDN: Iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) — Iterator protocol reference

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*