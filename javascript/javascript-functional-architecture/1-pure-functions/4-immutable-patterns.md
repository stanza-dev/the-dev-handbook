---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-immutable-patterns"
---

# Immutable Update Patterns

## Introduction

Complex state updates can be verbose with spread. Learn patterns and tools to make immutability practical.

## Key Concepts

**Spread Operator**: `{ ...obj, key: newVal }` creates a shallow copy with modifications.

**Reducer Pattern**: A pure function `(state, action) => newState` that returns a new state object.

**Lens**: A composable getter/setter pair for focusing on a specific part of a data structure.

**Immer**: A library that lets you write mutable-looking code inside `produce()` while generating immutable updates.

## Real World Context

Redux reducers, React's `useReducer`, and state management libraries all use immutable update patterns. Lenses are popular in Ramda and fp-ts for deeply nested state. Immer is used internally by Redux Toolkit to simplify reducer code.

## Deep Dive

### Helper Functions

```javascript
// Set nested property
function setIn(obj, path, value) {
  const [head, ...tail] = path;
  if (tail.length === 0) {
    return { ...obj, [head]: value };
  }
  return { ...obj, [head]: setIn(obj[head], tail, value) };
}

setIn(state, ['user', 'address', 'city'], 'LA');
```

### Using Immer

```javascript
import { produce } from 'immer';

const newState = produce(state, draft => {
  draft.user.address.city = 'LA';  // Looks mutable
  draft.items.push({ id: 1 });     // But creates new object
});
```

### Reducer Pattern

```javascript
function reducer(state, action) {
  switch (action.type) {
    case 'UPDATE_NAME':
      return { ...state, name: action.payload };
    case 'ADD_ITEM':
      return { ...state, items: [...state.items, action.payload] };
    default:
      return state;
  }
}
```

### Lens Pattern

```javascript
const nameLens = {
  get: obj => obj.name,
  set: (obj, val) => ({ ...obj, name: val })
};

nameLens.get(user);  // 'Alice'
nameLens.set(user, 'Bob');  // { name: 'Bob', ... }
```

## Common Pitfalls

1. **Spread nesting explosion** — Updating deeply nested state with spread creates verbose, error-prone code. Use Immer or lens libraries for deep updates.
2. **Forgetting to spread at every level** — `{ ...state, user: { name: 'Bob' } }` loses `user.email` if it existed. You need `{ ...state, user: { ...state.user, name: 'Bob' } }`.
3. **Creating too many intermediate objects** — Each spread creates a new object. For hot paths with many updates, consider batching or using Immer.

## Best Practices

1. **Flatten state shape** — Avoid deep nesting in state objects. Flatter structures need fewer spreads to update.
2. **Use helper functions for repeated patterns** — Extract `setIn(obj, path, value)` to avoid manual spread at every level.
3. **Choose the right tool for the complexity** — Spread for 1-2 levels deep, Immer for 3+, lenses for composable access patterns.

## Summary

Use helper functions for deep updates. Immer for complex state. Reducers for predictable updates. Lenses for composable access.

## Code Examples

**Immer's produce() lets you write mutable-looking code that produces immutable results — ideal for complex nested updates**

```javascript
import { produce } from 'immer';

const state = {
  user: { name: 'Alice', address: { city: 'NYC' } },
  items: [{ id: 1, name: 'Widget' }]
};

// With Immer: write mutable-looking code, get immutable result
const newState = produce(state, draft => {
  draft.user.address.city = 'LA';
  draft.items.push({ id: 2, name: 'Gadget' });
});

console.log(state.user.address.city); // 'NYC' — unchanged
console.log(newState.user.address.city); // 'LA'
```


## Resources

- [MDN: Spread Syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) — MDN reference on spread syntax for shallow copying objects and arrays

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*