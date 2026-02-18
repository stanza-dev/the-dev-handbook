---
source_course: "javascript"
source_lesson: "javascript-scope-closures"
---

# Scope & Closures

## Introduction

Scope determines where variables are accessible in your code. Closures—functions that remember their lexical scope—are one of JavaScript's most powerful features. Understanding scope and closures unlocks patterns like data privacy, factories, and memoization.

## Key Concepts

**Scope**: The context in which variables are declared and accessible.

**Lexical Scope**: Scope is determined by where variables are written in the code, not where functions are called.

**Closure**: A function that has access to variables from its outer (enclosing) scope, even after that outer function has returned.

## Real World Context

Closures enable module patterns (private variables), event handlers that remember state, React hooks, and curried functions. They're everywhere in JavaScript—understanding them is essential.

## Deep Dive

### Types of Scope

```javascript
// Global scope
const globalVar = 'I am global';

function outer() {
  // Function scope
  const outerVar = 'I am outer';
  
  if (true) {
    // Block scope (let/const only)
    let blockVar = 'I am block';
    var functionVar = 'I leak to function scope';
  }
  
  console.log(functionVar);  // 'I leak...'
  // console.log(blockVar);  // ReferenceError
}
```

### Scope Chain

```javascript
const a = 'global';

function outer() {
  const b = 'outer';
  
  function inner() {
    const c = 'inner';
    console.log(a, b, c);  // Can access all three
  }
  
  inner();
  // console.log(c);  // ReferenceError - can't go down
}
```

### Closures in Action

```javascript
function createCounter() {
  let count = 0;  // Private variable
  
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; }
  };
}

const counter = createCounter();
counter.increment();  // 1
counter.increment();  // 2
counter.getCount();   // 2
// count is private - can't access directly
```

### Closure in Loops

```javascript
// Classic problem with var
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Logs: 3, 3, 3 (all reference same i)

// Solution 1: Use let (block scope)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Logs: 0, 1, 2

// Solution 2: IIFE (old pattern)
for (var i = 0; i < 3; i++) {
  ((j) => {
    setTimeout(() => console.log(j), 100);
  })(i);
}
// Logs: 0, 1, 2
```

### Practical Closure Patterns

```javascript
// Factory function
function createGreeter(greeting) {
  return function(name) {
    return `${greeting}, ${name}!`;
  };
}

const sayHello = createGreeter('Hello');
const sayHi = createGreeter('Hi');
sayHello('Alice');  // 'Hello, Alice!'
sayHi('Bob');       // 'Hi, Bob!'

// Memoization
function memoize(fn) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    if (!(key in cache)) {
      cache[key] = fn.apply(this, args);
    }
    return cache[key];
  };
}

const expensiveFn = memoize((n) => {
  console.log('Computing...');
  return n * 2;
});
expensiveFn(5);  // 'Computing...' -> 10
expensiveFn(5);  // 10 (cached, no log)
```

## Common Pitfalls

1. **Closure in loops with var**: All callbacks share the same variable.
2. **Memory leaks**: Closures keep references to outer variables; be mindful with large objects.
3. **Unexpected `this`**: Remember closure captures variables, not `this` context.

## Best Practices

- **Use `let` in loops**: Avoids the classic closure problem.
- **Use closures for data privacy**: Instead of exposing internal state.
- **Be aware of memory**: Large closures can prevent garbage collection.
- **Name your functions**: Helps with debugging stack traces.

## Summary

Scope determines variable accessibility: global, function, and block scope. Closures are functions that retain access to their outer scope's variables. They enable powerful patterns like data privacy, factories, and memoization. Always use `let`/`const` in loops to avoid closure bugs.

## Resources

- [MDN: Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) — In-depth guide to closures
- [MDN: Scope](https://developer.mozilla.org/en-US/docs/Glossary/Scope) — Understanding JavaScript scope

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*