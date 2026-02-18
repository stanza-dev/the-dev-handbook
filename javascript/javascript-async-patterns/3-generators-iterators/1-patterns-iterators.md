---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-iterators"
---

# The Iterator Protocol

## Introduction

Iterators provide a standard way to traverse data structures. Any object implementing the iterator protocol can be used with for...of, spread operator, and destructuring. Understanding iterators unlocks powerful patterns for custom data structures.

## Key Concepts

**Iterator**: An object with a next() method returning {value, done}.

**Iterable**: An object with a [Symbol.iterator] method returning an iterator.

**Iterator Protocol**: The contract defining how iteration works.

## Real World Context

Arrays, Maps, Sets, Strings are all iterable. Custom iterables let you create lazy sequences, infinite streams, and composable data pipelines.

## Deep Dive

### Iterator Protocol

```javascript
// Manual iterator
const iterator = {
  current: 0,
  last: 5,
  next() {
    if (this.current <= this.last) {
      return { value: this.current++, done: false };
    }
    return { value: undefined, done: true };
  }
};

iterator.next(); // { value: 0, done: false }
iterator.next(); // { value: 1, done: false }
// ... until { value: undefined, done: true }
```

### Iterable Protocol

```javascript
const range = {
  start: 1,
  end: 5,
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
};

for (const num of range) {
  console.log(num);  // 1, 2, 3, 4, 5
}

[...range];  // [1, 2, 3, 4, 5]
const [first, second] = range;  // 1, 2
```

### Built-in Iterables

```javascript
// Arrays
for (const item of [1, 2, 3]) {}

// Strings
for (const char of 'hello') {}

// Maps (returns [key, value] pairs)
for (const [key, val] of new Map([['a', 1]])) {}

// Sets
for (const item of new Set([1, 2, 2, 3])) {}

// Getting iterators directly
const arr = [1, 2, 3];
arr.keys();     // Iterator of indices
arr.values();   // Iterator of values
arr.entries();  // Iterator of [index, value]
```

## Common Pitfalls

1. **Iterators are one-time use**: Once done, create a new one.
2. **Iterables vs Iterators**: Iterables have [Symbol.iterator], iterators have next().
3. **Infinite iterables**: Can crash spread operator; use with care.

## Best Practices

- **Implement [Symbol.iterator] for custom collections**: Enables for...of.
- **Return new iterator each call**: So collection can be iterated multiple times.
- **Use generators**: Easier than manual iterator implementation.

## Summary

Iterators have next() returning {value, done}. Iterables have [Symbol.iterator] returning an iterator. Built-in types like Array, String, Map, Set are iterable. Custom iterables work with for...of, spread, and destructuring.

## Resources

- [MDN: Iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) — Iterator and iterable protocols

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*