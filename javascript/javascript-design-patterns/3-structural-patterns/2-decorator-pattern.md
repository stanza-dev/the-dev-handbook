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

## Real World Context

Express/Koa middleware decorates request handlers. Logging, caching, rate-limiting, and authentication are all decorators wrapped around core logic. TC39 decorators (stage 3) bring native `@decorator` syntax to JavaScript classes.

## Deep Dive

### Function Decorator

A function decorator wraps an existing function to add behavior before and after the original call, without modifying the original:

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

The decorator uses `fn.apply(this, args)` to preserve the original function's `this` context and passes all arguments through unchanged.

### Memoization Decorator

Memoization caches function results keyed by arguments, turning repeated calls into Map lookups instead of recomputation:

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

The cache key is computed via `JSON.stringify(args)`, which works for primitive arguments. For object arguments, consider a more robust key strategy.

### Object Decorator

Object decorators use the spread operator to extend an existing object with new properties and methods:

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

The returned object has all original properties plus a `createdAt` timestamp and a `getAge()` method. The original object is unchanged, and the new one is a shallow copy.

## Common Pitfalls

1. **Order matters**: Decorators apply inside-out.
2. **Identity changes**: `decorated !== original`.
3. **`this` binding issues**: Use arrow functions or `.bind()`.

## Best Practices

1. **Keep decorators single-purpose** — Each decorator should add exactly one behavior (logging OR caching, not both).
2. **Preserve the original interface** — The decorated object should be a drop-in replacement for the original.
3. **Compose decorators explicitly** — Document the order in which decorators are applied, since order affects behavior.

## Summary

Decorators wrap functions/objects to add behavior. Use for logging, caching, validation. TC39 decorators provide native syntax for classes. Composition enables complex enhancements.

## Code Examples

**Function decorators for logging and timing — decorators compose inside-out, each wrapping the previous**

```javascript
function withLogging(fn) {
  return function(...args) {
    console.log(`Calling ${fn.name}(${args.join(', ')})`);
    const result = fn.apply(this, args);
    console.log(`→ ${result}`);
    return result;
  };
}

function withTiming(fn) {
  return function(...args) {
    const start = performance.now();
    const result = fn.apply(this, args);
    console.log(`${fn.name} took ${(performance.now() - start).toFixed(2)}ms`);
    return result;
  };
}

const add = (a, b) => a + b;
const enhanced = withLogging(withTiming(add));
enhanced(2, 3); // Logs timing and call info, returns 5
```


## Resources

- [Patterns.dev: Decorator Pattern](https://www.patterns.dev/posts/decorator-pattern/) — Decorator pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*