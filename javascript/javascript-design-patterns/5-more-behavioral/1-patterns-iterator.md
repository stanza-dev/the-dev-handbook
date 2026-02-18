---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-iterator"
---

# The Iterator Pattern

## Introduction

The Iterator pattern provides a way to access elements of a collection sequentially without exposing its underlying representation. JavaScript has this built in with the Iterator protocol.

## Deep Dive

### Custom Iterator

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

### Generator-Based Iterator

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

## Summary

Iterators provide sequential access to collections. JavaScript's Symbol.iterator enables for...of and spread. Generators make creating iterators simple.

## Resources

- [MDN: Iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) — Iterator protocol reference

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*