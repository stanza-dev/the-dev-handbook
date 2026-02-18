---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-currying"
---

# Currying

## Introduction

Currying transforms a function taking multiple arguments into a sequence of functions, each taking one argument. It enables partial application and more flexible composition.

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

## Summary

Currying transforms multi-arg functions into unary function chains. Enables partial application. Argument order matters—put data last.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*