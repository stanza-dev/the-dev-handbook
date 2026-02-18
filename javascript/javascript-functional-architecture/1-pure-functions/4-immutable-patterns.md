---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-immutable-patterns"
---

# Immutable Update Patterns

## Introduction

Complex state updates can be verbose with spread. Learn patterns and tools to make immutability practical.

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

## Summary

Use helper functions for deep updates. Immer for complex state. Reducers for predictable updates. Lenses for composable access.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*