---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-practical-fp"
---

# Practical FP in JavaScript

## Introduction

Functional programming in JavaScript doesn't require full purity. Mix FP where it helps, use objects/classes where they fit better.

## Deep Dive

### Using FP Libraries

```javascript
// Ramda - auto-curried, data-last
import { pipe, map, filter, prop } from 'ramda';

const getActiveUserNames = pipe(
  filter(prop('active')),
  map(prop('name'))
);

// fp-ts - TypeScript FP
import { pipe } from 'fp-ts/function';
import { map, filter } from 'fp-ts/Array';
import * as O from 'fp-ts/Option';
```

### Mixing Paradigms

```javascript
class UserService {
  constructor(api) {
    this.api = api;
  }
  
  // Methods use FP internally
  async getActiveUserNames() {
    const users = await this.api.fetchUsers();
    return pipe(
      users,
      filter(u => u.active),
      map(u => u.name)
    );
  }
}
```

### When to Use FP

```javascript
// Good for: Data transformation
const processOrders = pipe(
  filter(order => order.status === 'complete'),
  map(order => ({ ...order, total: calcTotal(order) })),
  sortBy(prop('date'))
);

// Consider classes for: Stateful services, complex domains
class ShoppingCart {
  #items = [];
  add(item) { /* ... */ }
  remove(id) { /* ... */ }
}
```

## Summary

Use FP for data transformations. Mix with OOP where appropriate. Libraries like Ramda, fp-ts help. Don't force purity everywhere.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*