---
source_course: "javascript-functional-architecture"
source_lesson: "javascript-functional-architecture-partial-application"
---

# Partial Application

## Introduction

Partial application fixes some arguments of a function, producing a new function with fewer parameters. Unlike currying, it can fix any number of arguments at once.

## Key Concepts

**Partial Application**: Fixing some arguments of a function upfront, returning a new function that takes the remaining arguments.

**bind()**: JavaScript's built-in partial application: `fn.bind(null, arg1, arg2)`.

**Partial from Right**: Fixing the last arguments instead of the first—useful when the data comes first.

## Real World Context

Event handlers often use partial application: `onClick={handleDelete.bind(null, itemId)}`. API client methods partially apply the base URL: `const getUser = get.bind(null, '/api/users')`. Configuration functions that fix options while leaving data flexible are everywhere in production code.

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

## Common Pitfalls

1. **Confusing currying with partial application** — Currying creates a chain of unary functions. Partial application fixes N arguments at once, regardless of arity.
2. **Losing `this` with bind** — `bind(null, ...)` sets `this` to `null`. If the function uses `this`, pass the correct context as the first argument.
3. **Over-specializing** — Partially applying too many arguments creates functions so specific they're only used once, adding complexity without reuse.

## Best Practices

1. **Use `bind` for simple cases** — `fn.bind(null, fixedArg)` is built-in and familiar to all JavaScript developers.
2. **Write a custom `partial` for flexibility** — A `partial(fn, ...presetArgs)` utility is clearer than `bind` for non-trivial cases.
3. **Combine with composition** — Partially applied functions are perfect building blocks for `pipe()` and `compose()` pipelines.

## Summary

Partial application fixes some arguments upfront. bind() for simple cases. Custom partial for flexibility. Different from currying in flexibility.

## Code Examples

**Partial application fixes some arguments upfront — both custom partial() and bind() achieve this**

```javascript
function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

function greet(greeting, name) {
  return `${greeting}, ${name}!`;
}

const sayHello = partial(greet, 'Hello');
sayHello('Alice'); // 'Hello, Alice!'
sayHello('Bob');   // 'Hello, Bob!'

// Using bind (built-in partial application)
const sayHi = greet.bind(null, 'Hi');
sayHi('Charlie');  // 'Hi, Charlie!'
```


## Resources

- [MDN: Function.prototype.bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) — MDN reference on bind() for partial application of function arguments

---

> 📘 *This lesson is part of the [Functional JavaScript Architecture](https://stanza.dev/courses/javascript-functional-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*