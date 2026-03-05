---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-memoization"
---

## Introduction

Every time a function is called with the same arguments, it performs the same computation and returns the same result. For expensive operations — recursive algorithms, complex data transformations, or repeated API lookups — this redundancy wastes CPU cycles. Memoization is the technique of caching function results so that subsequent calls with identical arguments return the cached value instantly instead of recomputing.

This pattern is fundamental to performance optimization in JavaScript, appearing everywhere from React's rendering optimizations to server-side request deduplication.

## Key Concepts

- **Memoization**: An optimization technique that stores the results of expensive function calls and returns the cached result when the same inputs occur again.
- **Cache Key**: The identifier used to look up cached results. For memoized functions, this is typically derived from the function arguments, often via JSON serialization or a custom hash.
- **Cache Invalidation**: The process of determining when cached results are stale and should be recomputed. This is famously one of the hardest problems in computer science.
- **LRU Cache**: A Least Recently Used cache eviction strategy that discards the least recently accessed entries when the cache reaches its size limit, balancing memory usage with hit rate.
- **Referential Equality**: In JavaScript, objects and arrays are compared by reference, not by value. Two objects with identical contents are not equal (`{} !== {}`), which affects cache key generation for non-primitive arguments.

## Real World Context

React's `React.memo()` memoizes component rendering based on prop equality. `useMemo` and `useCallback` memoize values and functions within components to prevent unnecessary recalculations and re-renders. On the server side, libraries like DataLoader batch and cache database queries to avoid the N+1 problem. GraphQL resolvers commonly use per-request memoization to deduplicate resolver calls. Fibonacci and dynamic programming algorithms rely on memoization to reduce exponential time complexity to linear.

## Deep Dive

The simplest memoization uses a `Map` to store results keyed by serialized arguments:

```javascript
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

For functions that accept objects as arguments, `WeakMap` provides cache entries that are automatically garbage collected when the key object is no longer referenced elsewhere:

```javascript
function memoizeWeak(fn) {
  const cache = new WeakMap();
  return function(obj) {
    if (cache.has(obj)) return cache.get(obj);
    const result = fn.call(this, obj);
    cache.set(obj, result);
    return result;
  };
}
```

For production use, an LRU cache prevents unbounded memory growth. JavaScript's `Map` maintains insertion order, which we can exploit: deleting and re-inserting a key moves it to the end, and the first key is always the oldest:

```javascript
class LRUCache {
  #max;
  #cache = new Map();
  constructor(max = 100) { this.#max = max; }
  get(key) {
    if (!this.#cache.has(key)) return undefined;
    const val = this.#cache.get(key);
    this.#cache.delete(key);
    this.#cache.set(key, val);
    return val;
  }
  set(key, val) {
    this.#cache.delete(key);
    this.#cache.set(key, val);
    if (this.#cache.size > this.#max)
      this.#cache.delete(this.#cache.keys().next().value);
  }
}
```

In React, `React.memo` wraps a component to skip re-rendering when props have not changed by shallow comparison. `useMemo(() => computeExpensive(a, b), [a, b])` caches a computed value and only recomputes when dependencies change. These are memoization at the framework level, applying the same underlying principle.

## Common Pitfalls

- **Unbounded cache growth**: A memoize function without a size limit will eventually consume all available memory if called with many unique argument combinations. Always use an LRU strategy or TTL-based expiration in production.
- **JSON.stringify for cache keys**: This fails silently for arguments containing functions, `undefined` values, circular references, or `Map`/`Set` instances. It also does not preserve key ordering in objects, potentially causing cache misses for equivalent inputs.
- **Memoizing impure functions**: Memoization assumes the function is pure — same inputs always produce same outputs. Memoizing a function that reads from mutable external state or has side effects will return stale or incorrect results.

## Best Practices

- Use `WeakMap` when memoizing functions that take object arguments to allow garbage collection of unused cache entries.
- Set explicit cache size limits using an LRU strategy to prevent memory leaks in long-running applications.
- In React, prefer `useMemo` for expensive computations and `React.memo` for components with stable props, but do not memoize everything — the overhead of memoization can exceed the cost of recomputation for simple operations.

## Summary

Memoization caches function results to avoid redundant computation. Simple implementations use `Map` with serialized keys, while `WeakMap` handles object arguments with automatic garbage collection. LRU caches bound memory usage in production. React provides built-in memoization through `React.memo`, `useMemo`, and `useCallback`. Always consider cache invalidation strategy and memory limits when implementing memoization.

## Code Examples

**Memoization and LRU cache implementation**

```javascript
// Simple memoization
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// LRU Cache with size limit
class LRUCache {
  #max;
  #cache = new Map();
  constructor(max = 100) { this.#max = max; }
  get(key) {
    if (!this.#cache.has(key)) return undefined;
    const val = this.#cache.get(key);
    this.#cache.delete(key);
    this.#cache.set(key, val); // Move to end
    return val;
  }
  set(key, val) {
    this.#cache.delete(key);
    this.#cache.set(key, val);
    if (this.#cache.size > this.#max)
      this.#cache.delete(this.#cache.keys().next().value);
  }
}
```


## Resources

- [MDN: Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) — Map object for efficient key-value caching

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*