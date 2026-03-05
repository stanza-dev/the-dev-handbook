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

## Real World Context

Promises are the most common monad in JavaScript—`.then()` is `flatMap`. Async/await is syntactic sugar for monadic promise chaining. Optional chaining (`a?.b?.c`) is a built-in Maybe monad operation. fp-ts, Effect, and neverthrow bring typed monads to TypeScript.

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

## Common Pitfalls

1. **Monad confusion** — Don't overthink it. If you've used `.then()` on a Promise, you've used `flatMap`. A monad is just a container with `flatMap`.
2. **Nesting instead of chaining** — Using `.map()` when you need `.flatMap()` creates nested containers: `Maybe(Maybe(x))` instead of `Maybe(x)`.
3. **Mixing monad types** — You can't directly chain a `Maybe` into an `Either`. Use natural transformations (converters) between monad types.

## Best Practices

1. **Use `flatMap` when the mapping function returns a container** — `flatMap` = map + flatten. If `fn` returns a raw value, use `map`. If it returns a container, use `flatMap`.
2. **Think in pipelines** — `Maybe(x).flatMap(f).flatMap(g).fold(...)` reads like a pipeline where any step can short-circuit.
3. **Start with Promise patterns** — If you understand `.then()` chaining, you understand monads. Generalize from there.

## Summary

Monads have flatMap for chaining container-returning functions. Prevents nesting. Promises are monads. flatMap = map + flatten.

## Code Examples

**Monads extend functors with flatMap — it prevents container nesting when chaining operations that return containers**

```javascript
// The nesting problem: map with container-returning functions
Maybe(user)
  .map(u => Maybe(u.address))  // Maybe(Maybe(address)) — nested!

// Solution: flatMap = map + flatten
Maybe(user)
  .flatMap(u => Maybe(u.address))  // Maybe(address) — flat!
  .flatMap(a => Maybe(a.street))   // Maybe(street)
  .fold(() => 'Unknown', s => s);

// Promise.then IS flatMap
fetchUser(id)
  .then(user => fetchPosts(user.id))  // Returns Promise — auto-flattened
  .then(posts => posts[0]);
```


## Resources

- [MDN: Promise.prototype.then() — Promise as Monad](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) — Official documentation: MDN: Promise.prototype.then() — Promise as Monad

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*