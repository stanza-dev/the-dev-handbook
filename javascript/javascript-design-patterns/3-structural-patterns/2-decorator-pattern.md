---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-decorator-pattern"
---

# The Decorator Pattern

## Introduction

The Decorator pattern dynamically adds behavior to objects without modifying their class. It wraps objects to extend functionality while keeping the same interface.

## Key Concepts

**Decorator**: Wrapper that adds functionality.

**Component**: The original object being decorated.

**Transparent Wrapping**: Same interface as the wrapped object.

## Deep Dive

### Function Decorator

```javascript
function withLogging(fn) {
  return function(...args) {
    console.log(`Calling ${fn.name}`);
    const result = fn.apply(this, args);
    console.log(`Result: ${result}`);
    return result;
  };
}

function add(a, b) { return a + b; }
const loggedAdd = withLogging(add);
loggedAdd(2, 3);  // Logs call and result
```

### Memoization Decorator

```javascript
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const expensiveCalc = memoize((n) => {
  // Heavy computation
  return n * n;
});
```

### Object Decorator

```javascript
function withTimestamp(obj) {
  return {
    ...obj,
    createdAt: Date.now(),
    getAge() { return Date.now() - this.createdAt; }
  };
}

const user = withTimestamp({ name: 'Alice' });
```

## Common Pitfalls

1. **Order matters**: Decorators apply inside-out.
2. **Identity changes**: `decorated !== original`.
3. **`this` binding issues**: Use arrow functions or `.bind()`.

## Summary

Decorators wrap functions/objects to add behavior. Use for logging, caching, validation. TC39 decorators provide native syntax for classes. Composition enables complex enhancements.

## Resources

- [Patterns.dev: Decorator Pattern](https://www.patterns.dev/posts/decorator-pattern/) — Decorator pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*