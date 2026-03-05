---
source_course: "javascript"
source_lesson: "javascript-variables-and-types"
---

# Variables & Data Types

## Introduction

Every program needs to store and manipulate data. In JavaScript, variables are containers that hold values, and understanding how they work is the foundation of everything you'll build. Unlike statically typed languages, JavaScript figures out types at runtime—a double-edged sword that offers flexibility but requires discipline.

## Key Concepts

**Variable**: A named storage location for data. Declared using `let`, `const`, or the legacy `var`.

**Data Type**: The classification of data that tells the interpreter how to handle it. JavaScript has **primitives** (immutable values) and **objects** (mutable collections).

**Dynamic Typing**: Variables can hold any type and can change types during execution.

## Real World Context

In production applications, poor variable management leads to bugs that are hard to trace. Using `const` by default prevents accidental reassignments. Understanding the difference between primitives and objects is critical when passing data to functions—primitives are copied by value, objects by reference.

## Deep Dive

### Declaration Keywords

Modern JavaScript uses `let` and `const`. Avoid `var` as it has function scope rather than block scope.

```javascript
const API_URL = 'https://api.example.com'; // Cannot be reassigned
let retryCount = 0; // Can be reassigned
retryCount = 1; // OK
// API_URL = 'new-url'; // TypeError!
```

### The 7 Primitive Types

1. **string** - Text data: `"hello"`
2. **number** - Integers and floats: `42`, `3.14`
3. **bigint** - Arbitrary precision integers: `9007199254740991n`
4. **boolean** - `true` or `false`
5. **undefined** - Variable declared but not assigned
6. **symbol** - Unique identifiers: `Symbol('id')`
7. **null** - Intentional absence of value

Everything else (arrays, functions, objects) is an **Object**.

```javascript
typeof 'hello';    // 'string'
typeof 42;         // 'number'
typeof true;       // 'boolean'
typeof undefined;  // 'undefined'
typeof Symbol();   // 'symbol'
typeof null;       // 'object' (historical bug!)
typeof {};         // 'object'
typeof [];         // 'object'
```

## Common Pitfalls

1. **Using `var` in loops**: Variables declared with `var` are hoisted and shared across iterations, causing unexpected closure behavior.
2. **Confusing `null` and `undefined`**: Use `undefined` for uninitialized variables; use `null` to explicitly indicate "no value".
3. **Assuming `typeof null === 'null'`**: This returns `'object'` due to a legacy bug in JavaScript.

## Best Practices

- **Default to `const`**: Only use `let` when you need to reassign. Never use `var`.
- **Use meaningful names**: `userAge` is better than `x`.
- **Initialize variables**: Avoid leaving variables as `undefined` when possible.
- **Use strict equality (`===`)**: Avoids type coercion surprises.

## Summary

JavaScript variables are declared with `const` (immutable binding) or `let` (mutable binding). The language has 7 primitive types and objects. Understanding the difference between primitives (passed by value) and objects (passed by reference) is fundamental to avoiding bugs.

## Code Examples

**Variable declaration with const and let**

```javascript
const API_URL = 'https://api.example.com'; // Cannot be reassigned
let retryCount = 0; // Can be reassigned
retryCount = 1; // OK
// API_URL = 'new-url'; // TypeError!
```

**The typeof operator and JavaScript's 7 primitive types**

```javascript
typeof 'hello';    // 'string'
typeof 42;         // 'number'
typeof true;       // 'boolean'
typeof undefined;  // 'undefined'
typeof Symbol();   // 'symbol'
typeof null;       // 'object' (historical bug!)
typeof {};         // 'object'
typeof [];         // 'object'
```


## Resources

- [MDN: Grammar and Types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types) — Official guide to JavaScript syntax and data types
- [MDN: JavaScript Data Structures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures) — Comprehensive overview of all JavaScript data types

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*