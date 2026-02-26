---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-practical-fp"
---

# Practical FP in JavaScript

## Introduction

Functional programming in JavaScript doesn't require full purity. Mix FP where it helps, use objects/classes where they fit better.

## Key Concepts

**Pragmatic FP**: Using functional patterns where they help, without dogmatic purity.

**Functional Core, Imperative Shell**: Pure business logic surrounded by effectful I/O boundaries.

**FP Libraries**: Ramda (auto-curried, data-last), fp-ts (typed FP for TypeScript), Immer (immutable state).

## Real World Context

Most production JavaScript is a mix of FP and OOP. React components are functional but use class-based error boundaries. Redux reducers are pure functions called by an imperative dispatch system. The goal is to use the right tool for each problem, not to be 100% functional.

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

## Common Pitfalls

1. **Forcing purity everywhere** — I/O, DOM manipulation, and timers are inherently impure. Isolate them, don't eliminate them.
2. **Library dependency for simple patterns** — You don't need Ramda for `arr.map(x => x.name)`. Use libraries when they add real value.
3. **Readability trade-offs** — A point-free pipeline with 8 composed functions is harder to maintain than explicit code. Optimize for the team's understanding.

## Best Practices

1. **Use FP for data transformation** — `pipe(filter, map, reduce)` is cleaner than nested loops for data processing.
2. **Use classes for stateful services** — Database connections, WebSocket managers, and caches are naturally stateful. Use classes.
3. **Start small** — Introduce FP patterns incrementally: pure functions first, then composition, then functors/monads as needed.

## Summary

Use FP for data transformations. Mix with OOP where appropriate. Libraries like Ramda, fp-ts help. Don't force purity everywhere.

## Code Examples

**Practical FP mixes paradigms — functional core for data transforms, imperative shell for effects and services**

```javascript
import { pipe, filter, map, prop } from 'ramda';

// FP for data transformation
const getActiveUserNames = pipe(
  filter(prop('active')),
  map(prop('name'))
);

// OOP for stateful services
class UserService {
  constructor(api) { this.api = api; }

  async getActiveNames() {
    const users = await this.api.fetchUsers(); // Imperative shell
    return getActiveUserNames(users);           // Functional core
  }
}

// Mix paradigms pragmatically
const service = new UserService(apiClient);
const names = await service.getActiveNames();
```


## Resources

- [MDN: Array Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) — MDN overview of functional array methods used in practical FP patterns

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*