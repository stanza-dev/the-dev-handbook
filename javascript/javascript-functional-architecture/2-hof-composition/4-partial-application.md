---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-partial-application"
---

# Partial Application

## Introduction

Partial application fixes some arguments of a function, producing a new function with fewer parameters. Unlike currying, it can fix any number of arguments at once.

## Deep Dive

### Using bind()

```javascript
function greet(greeting, name) {
  return `${greeting}, ${name}!`;
}

const sayHello = greet.bind(null, 'Hello');
sayHello('Alice');  // 'Hello, Alice!'
sayHello('Bob');    // 'Hello, Bob!'
```

### Partial Function

```javascript
function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

const sayHello = partial(greet, 'Hello');
sayHello('Alice');  // 'Hello, Alice!'

// Multiple preset args
const add = (a, b, c) => a + b + c;
const add5and10 = partial(add, 5, 10);
add5and10(15);  // 30
```

### Partial from Right

```javascript
function partialRight(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...laterArgs, ...presetArgs);
  };
}

const divide = (a, b) => a / b;
const divideBy2 = partialRight(divide, 2);
divideBy2(10);  // 5
```

### Currying vs Partial Application

```javascript
// Currying: unary chain
const curried = a => b => c => a + b + c;
curried(1)(2)(3);  // 6

// Partial: fix any number
const partial = partial(add, 1, 2);
partial(3);  // 6
```

## Summary

Partial application fixes some arguments upfront. bind() for simple cases. Custom partial for flexibility. Different from currying in flexibility.

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*