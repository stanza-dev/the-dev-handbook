---
source_course: "javascript"
source_lesson: "javascript-arrays-basics"
---

# Arrays: Creation & Access

## Introduction

Arrays are ordered collections that store multiple values in a single variable. They're the workhorse of data manipulation in JavaScript—from storing user lists to processing API responses. Understanding arrays deeply is essential for every JavaScript developer.

## Key Concepts

**Array**: An ordered, indexed collection of values. Zero-indexed (first element is at index 0).

**Sparse Array**: An array with gaps (missing indices). Generally avoided.

**Array-like Object**: Objects with numeric keys and a `length` property but lacking array methods (e.g., `arguments`, NodeList).

## Real World Context

Arrays hold product lists, user records, search results, form inputs, and virtually any collection of data. They're the foundation for rendering lists in UI frameworks and processing data from APIs.

## Deep Dive

### Creating Arrays

```javascript
// Array literal (preferred)
const fruits = ['apple', 'banana', 'cherry'];

// Array constructor (rarely used)
const empty = new Array(5);  // Creates array with 5 empty slots
const nums = new Array(1, 2, 3);  // [1, 2, 3]

// Array.of (consistent behavior)
Array.of(5);  // [5] (not 5 empty slots)

// Array.from (convert iterables/array-likes)
Array.from('hello');  // ['h', 'e', 'l', 'l', 'o']
Array.from({ length: 3 }, (_, i) => i);  // [0, 1, 2]
```

### Accessing Elements

```javascript
const arr = ['a', 'b', 'c', 'd', 'e'];

arr[0];      // 'a' (first element)
arr[arr.length - 1];  // 'e' (last element)
arr.at(-1);  // 'e' (ES2022: negative indexing)
arr.at(-2);  // 'd'

// Destructuring
const [first, second, ...rest] = arr;
// first = 'a', second = 'b', rest = ['c', 'd', 'e']
```

### Array Properties

```javascript
const arr = [1, 2, 3];

arr.length;  // 3
Array.isArray(arr);  // true
Array.isArray('not array');  // false
```

### Checking Contents

```javascript
const fruits = ['apple', 'banana', 'cherry'];

fruits.includes('banana');  // true
fruits.includes('grape');   // false
fruits.indexOf('banana');   // 1
fruits.indexOf('grape');    // -1 (not found)
```

## Common Pitfalls

1. **Accessing out-of-bounds**: Returns `undefined`, not an error. Can mask bugs.
2. **Confusing length with last index**: Last index is `length - 1`.
3. **Using `typeof` on arrays**: Returns `'object'`. Use `Array.isArray()` instead.

## Best Practices

- **Use array literals `[]`**: Not the `Array` constructor.
- **Use `Array.isArray()`**: For type checking arrays.
- **Use `.at(-1)` for last element**: Cleaner than `arr[arr.length - 1]`.
- **Prefer destructuring**: For extracting values from arrays.

## Summary

Arrays are ordered, zero-indexed collections. Create them with literals (`[]`), access elements with brackets or `.at()`, check contents with `includes()` or `indexOf()`. Use `Array.isArray()` for type checking and destructuring for elegant value extraction.

## Resources

- [MDN: Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) — Complete Array reference
- [MDN: Array.from()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from) — Creating arrays from iterables

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*