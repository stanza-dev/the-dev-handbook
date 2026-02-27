---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-memoization"
---

# Memoization

## Introduction

Memoization caches function results based on arguments. When called with the same arguments, it returns the cached result instead of recomputing.

## Key Concepts

**Cache Key**: How arguments are serialized for lookup.

**Cache Invalidation**: When to clear cached results.

**Pure Requirement**: Only works reliably with pure functions.

## Deep Dive

### Basic Implementation

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

// Usage
const expensiveCalc = memoize((n) => {
  console.log('Computing...');
  return n * n;
});

expensiveCalc(5);  // 'Computing...' → 25
expensiveCalc(5);  // 25 (cached, no log)
```

### Memoized Fibonacci

```javascript
const fib = memoize((n) => {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
});

fib(50);  // Instant! Without memo: ~minutes
```

### LRU Memoization

```javascript
function memoizeLRU(fn, maxSize = 100) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      const value = cache.get(key);
      cache.delete(key);
      cache.set(key, value);  // Move to end
      return value;
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    if (cache.size > maxSize) {
      cache.delete(cache.keys().next().value);
    }
    return result;
  };
}
```

## Common Pitfalls

1. **Object arguments**: JSON.stringify order isn't guaranteed.
2. **Memory leaks**: Unbounded caches grow forever.
3. **Impure functions**: Cache becomes invalid.

## Summary

Memoization trades memory for speed. Use for expensive pure functions. Consider LRU cache to bound memory. Only works with pure functions.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*