---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-fp-utilities"
---

# Essential FP Utilities

## Introduction

A toolkit of small, composable functions makes functional programming practical.

## Key Concepts

**Identity**: `x => x` — returns input unchanged. Useful as a default transformer.

**Constant**: `x => () => x` — always returns the same value, ignoring its argument.

**Prop/Path**: Accessors that extract values from objects by key or nested path.

**Tap**: `fn => x => { fn(x); return x; }` — executes a side effect and passes the value through unchanged.

## Real World Context

Ramda, lodash/fp, and Sanctuary all export these core utilities. `identity` is used as a default transform in mapping functions. `prop('name')` replaces `user => user.name` across codebases. `tap` enables logging in functional pipelines without breaking the chain.

## Deep Dive

### Identity and Constant

```javascript
const identity = x => x;
const constant = x => () => x;

// Use identity for default transforms
const transform = fn || identity;

// Use constant for fixed values
map.defaultValue(constant(0));
```

### Property Access

```javascript
const prop = key => obj => obj[key];
const path = keys => obj => keys.reduce((o, k) => o?.[k], obj);

users.map(prop('name'));
orders.map(path(['customer', 'address', 'city']));
```

### Predicates

```javascript
const not = fn => (...args) => !fn(...args);
const both = (f, g) => (...args) => f(...args) && g(...args);
const either = (f, g) => (...args) => f(...args) || g(...args);

const isEven = n => n % 2 === 0;
const isPositive = n => n > 0;

const isPositiveEven = both(isEven, isPositive);
numbers.filter(isPositiveEven);
```

### Tap for Debugging

```javascript
const tap = fn => x => { fn(x); return x; };

data
  .map(transform)
  .filter(tap(console.log))  // Log but pass through
  .reduce(combine);
```

### Object.groupBy (ES2024)

`Object.groupBy()` replaces the common reduce-based grouping pattern with a built-in utility.

```javascript
const users = [
  { name: 'Alice', role: 'admin' },
  { name: 'Bob', role: 'user' },
  { name: 'Charlie', role: 'admin' }
];
const grouped = Object.groupBy(users, u => u.role);
// { admin: [{ name: 'Alice', ... }, { name: 'Charlie', ... }], user: [{ name: 'Bob', ... }] }
```

## Common Pitfalls

1. **Overusing point-free utilities** — `compose(map(prop('name')), filter(both(isActive, not(isAdmin))))` can be harder to read than explicit arrow functions.
2. **Forgetting null safety in prop/path** — `prop('name')(undefined)` throws. Use optional chaining or Maybe for safety.
3. **Tap side effects in production** — Leaving `tap(console.log)` in production code is a common mistake. Use it for debugging only.

## Best Practices

1. **Build a small utility library** — Start with `identity`, `constant`, `prop`, `path`, `tap`, `pipe`, `compose`. Add utilities as needed.
2. **Use pipe/compose to combine utilities** — `const getName = pipe(prop('user'), prop('name'))` is clean and composable.
3. **Type your utilities** — In TypeScript, generic types on `prop<K>()` and `path()` provide autocomplete and catch errors.

## Summary

Build a library of small utilities. identity, constant, prop, path are essential. Predicates compose with both/either/not. tap helps debugging.

## Code Examples

**Essential FP utilities — identity, constant, prop, path, tap, and predicate combinators compose into expressive data processing**

```javascript
// Core FP utilities
const identity = x => x;
const constant = x => () => x;
const prop = key => obj => obj[key];
const path = keys => obj => keys.reduce((o, k) => o?.[k], obj);
const tap = fn => x => { fn(x); return x; };
const not = fn => (...args) => !fn(...args);
const both = (f, g) => (...args) => f(...args) && g(...args);

// Usage
const users = [
  { name: 'Alice', active: true, age: 30 },
  { name: 'Bob', active: false, age: 25 }
];

const isActive = prop('active');
const isAdult = u => u.age >= 18;

users
  .filter(both(isActive, isAdult))
  .map(prop('name')); // ['Alice']
```


## Resources

- [MDN: Optional Chaining (?.)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining) — MDN reference on optional chaining for safe property access

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*