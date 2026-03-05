---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-currying"
---

# Currying

## Introduction

Currying transforms a function taking multiple arguments into a sequence of functions, each taking one argument. It enables partial application and more flexible composition.

## Key Concepts

**Currying**: Transforming `f(a, b, c)` into `f(a)(b)(c)` — a chain of unary functions.

**Partial Application**: Fixing some arguments of a function, creating a new function with fewer parameters.

**Data-Last**: Placing the data argument last so curried functions compose naturally (e.g., `map(fn)(data)`).

## Real World Context

Ramda and lodash/fp are entirely built on auto-curried, data-last functions. React's higher-order components use currying: `withAuth(Component)` returns a new component. Event handler factories like `handleChange(field)(event)` are curried patterns used daily in React forms.

## Deep Dive

### Basic Currying

```javascript
// Uncurried
function add(a, b, c) {
  return a + b + c;
}
add(1, 2, 3);  // 6

// Curried
const addCurried = a => b => c => a + b + c;
addCurried(1)(2)(3);  // 6

// Partial application
const add1 = addCurried(1);
const add1and2 = add1(2);
add1and2(3);  // 6
```

### Generic Curry Function

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

const curriedAdd = curry((a, b, c) => a + b + c);
curriedAdd(1)(2)(3);    // 6
curriedAdd(1, 2)(3);    // 6
curriedAdd(1)(2, 3);    // 6
```

### Practical Uses

```javascript
const map = curry((fn, arr) => arr.map(fn));
const filter = curry((pred, arr) => arr.filter(pred));

const double = map(x => x * 2);
const evens = filter(x => x % 2 === 0);

double([1, 2, 3]);   // [2, 4, 6]
evens([1, 2, 3, 4]); // [2, 4]
```

## Common Pitfalls

1. **Argument order matters** — Put configuration first, data last. `map(fn)(arr)` composes; `map(arr)(fn)` doesn't.
2. **Debugging curried chains** — Stack traces with anonymous curried functions are hard to read. Name your intermediate functions.
3. **Variadic functions can't be curried** — `Math.max(...args)` has no fixed arity. Currying requires knowing how many arguments to expect.

## Best Practices

1. **Curry configuration, not data** — `createLogger(level)(message)` makes sense; `createLogger(message)(level)` doesn't.
2. **Use auto-curry utilities** — A generic `curry()` function avoids manually writing `a => b => c => ...` chains.
3. **Combine with pipe/compose** — Curried functions are building blocks for pipelines: `pipe(map(double), filter(isEven))`.

## Summary

Currying transforms multi-arg functions into unary function chains. Enables partial application. Argument order matters—put data last.

## Code Examples

**Generic curry function that works with any arity — supports both full and partial argument passing**

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

const add = curry((a, b, c) => a + b + c);
add(1)(2)(3);    // 6
add(1, 2)(3);    // 6
add(1)(2, 3);    // 6

const map = curry((fn, arr) => arr.map(fn));
const double = map(x => x * 2);
double([1, 2, 3]); // [2, 4, 6]
```


## Resources

- [MDN: Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) — MDN guide to closures — the mechanism that makes currying and partial application work

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*