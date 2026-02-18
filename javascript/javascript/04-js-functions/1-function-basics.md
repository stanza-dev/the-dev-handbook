---
source_course: "javascript"
source_lesson: "javascript-function-basics"
---

# Function Declarations & Expressions

## Introduction

Functions are the building blocks of JavaScript—reusable pieces of code that perform specific tasks. Understanding the different ways to define functions and their subtle differences is fundamental to writing clean, predictable code.

## Key Concepts

**Function Declaration**: A named function defined with the `function` keyword. Hoisted to the top of its scope.

**Function Expression**: A function assigned to a variable. Not hoisted.

**Hoisting**: JavaScript's behavior of moving declarations to the top of their scope before execution.

## Real World Context

Functions organize code into logical units, enable reuse, and form the basis of APIs and modules. Understanding hoisting prevents bugs when functions are called before they appear in the code.

## Deep Dive

### Function Declaration

```javascript
// Hoisted - can be called before definition
greet('Alice');  // Works!

function greet(name) {
  return `Hello, ${name}!`;
}
```

### Function Expression

```javascript
// Not hoisted - must be defined before use
// sayHi('Bob');  // ReferenceError!

const sayHi = function(name) {
  return `Hi, ${name}!`;
};

sayHi('Bob');  // Works!
```

### Named Function Expressions

```javascript
const factorial = function fact(n) {
  if (n <= 1) return 1;
  return n * fact(n - 1);  // Can reference itself by name
};

factorial(5);  // 120
// fact(5);    // ReferenceError - name only visible inside
```

### Parameters and Arguments

```javascript
// Default parameters (ES6)
function greet(name = 'Guest', greeting = 'Hello') {
  return `${greeting}, ${name}!`;
}
greet();           // 'Hello, Guest!'
greet('Alice');    // 'Hello, Alice!'

// Rest parameters
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4);  // 10

// Arguments object (avoid in modern code)
function oldSum() {
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}
```

### Return Values

```javascript
// Explicit return
function add(a, b) {
  return a + b;
}

// No return = undefined
function noReturn() {
  console.log('Side effect');
}
noReturn();  // undefined

// Early return pattern
function divide(a, b) {
  if (b === 0) return null;  // Guard clause
  return a / b;
}
```

## Common Pitfalls

1. **Calling expression before definition**: Unlike declarations, expressions aren't hoisted.
2. **Forgetting return**: Functions without return give `undefined`.
3. **Modifying arguments**: The `arguments` object is not a real array and has quirks.

## Best Practices

- **Prefer const with function expressions**: `const fn = function() {}` prevents reassignment.
- **Use default parameters**: Instead of checking for undefined inside the function.
- **Use rest parameters over arguments**: `...args` gives a real array.
- **Keep functions small**: One function, one responsibility.

## Summary

Function declarations are hoisted and can be called anywhere in their scope. Function expressions are not hoisted and must be defined before use. Use default parameters and rest parameters for flexible APIs. Functions without an explicit return statement return `undefined`.

## Resources

- [MDN: Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions) — Comprehensive guide to JavaScript functions
- [MDN: Default Parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters) — Default parameter values

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*