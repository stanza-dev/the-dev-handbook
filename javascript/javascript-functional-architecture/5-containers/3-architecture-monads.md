---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-monads"
---

# Monads & flatMap

## Introduction

Monads extend functors with flatMap (chain). This allows chaining operations that return containers, avoiding nested containers.

## Key Concepts

**flatMap/chain**: Map then flatten nested container.

**Monad Laws**: Left identity, right identity, associativity.

## Deep Dive

### The Nesting Problem

```javascript
// Without flatMap
Maybe(user)
  .map(u => Maybe(u.address))  // Maybe(Maybe(address))
  .map(m => m.map(a => a.street));  // Nested!

// With flatMap
Maybe(user)
  .flatMap(u => Maybe(u.address))  // Maybe(address)
  .flatMap(a => Maybe(a.street));  // Maybe(street)
```

### Implementing flatMap

```javascript
const Just = x => ({
  map: f => Just(f(x)),
  flatMap: f => f(x),  // f returns a Maybe
  fold: (onNothing, onJust) => onJust(x)
});

const Nothing = () => ({
  map: f => Nothing(),
  flatMap: f => Nothing(),  // Short-circuit
  fold: (onNothing, onJust) => onNothing()
});
```

### Promise is a Monad

```javascript
// Promise.then is flatMap
fetchUser(id)
  .then(user => fetchPosts(user.id))  // Returns Promise
  .then(posts => posts[0]);            // Flattened

// .then handles both map and flatMap!
```

### Chaining Operations

```javascript
const getStreetName = user =>
  Maybe(user)
    .flatMap(u => Maybe(u.address))
    .flatMap(a => Maybe(a.street))
    .flatMap(s => Maybe(s.name))
    .fold(() => 'Unknown', name => name);
```

## Summary

Monads have flatMap for chaining container-returning functions. Prevents nesting. Promises are monads. flatMap = map + flatten.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*