---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-immutability"
---

# Immutability

## Introduction

Immutability means never modifying data after creation. Instead, create new versions with changes. This eliminates entire classes of bugs.

## Key Concepts

**Immutability**: Never modifying existing data—always creating new copies with changes.

**Shallow Copy**: Copying only the top-level properties (`{ ...obj }`). Nested objects are still shared.

**Deep Copy**: Recursively copying all levels. The built-in `structuredClone()` API (available since 2022) provides this natively.

**Structural Sharing**: Reusing unchanged parts of a data structure in the new version (used by Immer, Immutable.js).

## Real World Context

React relies on immutability for efficient re-rendering—it uses reference equality (`===`) to detect changes. Redux requires immutable state updates for its time-travel debugger. The newer `Set` methods (`union()`, `intersection()`, `difference()`) all return new Sets rather than mutating, following the immutability principle.

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

### Immutable Set Operations (ES2025)

New `Set` methods return new Sets instead of mutating, following the immutability principle.

```javascript
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);
setA.union(setB);        // Set {1, 2, 3, 4}
setA.intersection(setB); // Set {2, 3}
setA.difference(setB);   // Set {1}
// setA is unchanged — all methods return new Sets
```

## Common Pitfalls

1. **Forgetting spread is shallow** — `{ ...obj }` copies top-level properties but nested objects are still shared references. Use `structuredClone()` for deep copies.
2. **Object.freeze is shallow** — `Object.freeze({ inner: { x: 1 } })` does not freeze `inner`. Nested objects remain mutable.
3. **Array methods that mutate** — `push()`, `pop()`, `splice()`, `sort()`, `reverse()` all mutate. Use `concat()`, `filter()`, `toSorted()`, `toReversed()` instead.

## Best Practices

1. **Use `structuredClone()` for deep copies** — This built-in API (available since 2022) handles nested objects, Maps, Sets, Dates, and more.
2. **Prefer non-mutating array methods** — Use `toSorted()`, `toReversed()`, `toSpliced()`, `with()` (ES2023) for clean immutable array operations.
3. **Use Immer for complex state updates** — When spread nesting gets deep, Immer's `produce()` lets you write mutable-looking code that produces immutable results.

## Summary

Never mutate—create new versions. Use spread for objects/arrays. Object.freeze for enforcement. Consider Immer for complex updates.

## Code Examples

**Immutable updates using spread and structuredClone — original data is never modified**

```javascript
const user = { name: 'Alice', address: { city: 'NYC' } };

// Shallow copy with spread
const updated = { ...user, name: 'Bob' };

// Deep copy with structuredClone (built-in since 2022)
const deep = structuredClone(user);
deep.address.city = 'LA';
console.log(user.address.city); // 'NYC' — original untouched

// Immutable array operations
const arr = [1, 2, 3];
const added = [...arr, 4];           // [1, 2, 3, 4]
const removed = arr.filter(x => x !== 2); // [1, 3]
```


## Resources

- [MDN: structuredClone()](https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone) — Official MDN documentation for the structuredClone() deep cloning API
- [MDN: Object.freeze()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) — Official MDN documentation for Object.freeze() to prevent object mutation

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*