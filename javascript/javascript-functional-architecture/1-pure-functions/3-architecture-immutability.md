---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-immutability"
---

# Immutability

## Introduction

Immutability means never modifying data after creation. Instead, create new versions with changes. This eliminates entire classes of bugs.

## Deep Dive

### Object Immutability

```javascript
// Mutable (bad)
const user = { name: 'Alice', age: 25 };
user.age = 26;  // Mutates original

// Immutable (good)
const updatedUser = { ...user, age: 26 };  // New object

// Nested updates
const state = {
  user: { name: 'Alice', address: { city: 'NYC' } }
};

const newState = {
  ...state,
  user: {
    ...state.user,
    address: {
      ...state.user.address,
      city: 'LA'
    }
  }
};
```

### Array Immutability

```javascript
const arr = [1, 2, 3];

// Instead of push
const added = [...arr, 4];

// Instead of splice
const removed = arr.filter((_, i) => i !== 1);

// Instead of direct assignment
const updated = arr.map((x, i) => i === 1 ? 10 : x);
```

### Object.freeze

```javascript
const frozen = Object.freeze({ a: 1 });
frozen.a = 2;  // Silently fails (or throws in strict)
frozen.a;  // 1

// Shallow freeze only!
const nested = Object.freeze({ inner: { x: 1 } });
nested.inner.x = 2;  // Works! Inner not frozen
```

## Summary

Never mutate—create new versions. Use spread for objects/arrays. Object.freeze for enforcement. Consider Immer for complex updates.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*