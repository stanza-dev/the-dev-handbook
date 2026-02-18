---
source_course: "javascript"
source_lesson: "javascript-iterators-for-of-in"
---

# Iterators: for...of and for...in

## Introduction

ES6 introduced `for...of`, a cleaner way to iterate over iterable objects. Combined with the older `for...in` (for object properties), you have powerful tools for traversing any data structure. Understanding when to use each prevents common bugs.

## Key Concepts

**Iterable**: An object that implements the iteration protocol (arrays, strings, Maps, Sets, etc.).

**for...of**: Iterates over **values** of an iterable.

**for...in**: Iterates over **enumerable property keys** of an object.

## Real World Context

Processing API response arrays, iterating over Map entries, reading characters from strings, traversing Set values—`for...of` handles all these elegantly. `for...in` is useful for inspecting object properties dynamically, like when building form data from an object.

## Deep Dive

### for...of (Iterate Values)

```javascript
// Arrays
const colors = ['red', 'green', 'blue'];
for (const color of colors) {
  console.log(color);  // 'red', 'green', 'blue'
}

// Strings
for (const char of 'Hello') {
  console.log(char);  // 'H', 'e', 'l', 'l', 'o'
}

// Maps
const map = new Map([['a', 1], ['b', 2]]);
for (const [key, value] of map) {
  console.log(key, value);  // 'a' 1, 'b' 2
}

// Sets
const set = new Set([1, 2, 3]);
for (const num of set) {
  console.log(num);  // 1, 2, 3
}
```

### for...in (Iterate Keys)

```javascript
const user = { name: 'Alice', age: 30, city: 'NYC' };
for (const key in user) {
  console.log(key, user[key]);  // 'name' 'Alice', 'age' 30, 'city' 'NYC'
}
```

### Getting Index with for...of

```javascript
const fruits = ['apple', 'banana', 'cherry'];

// Using entries()
for (const [index, fruit] of fruits.entries()) {
  console.log(index, fruit);  // 0 'apple', 1 'banana', 2 'cherry'
}
```

### The Danger of for...in with Arrays

```javascript
const arr = ['a', 'b', 'c'];
arr.customProp = 'oops';

// for...in iterates ALL enumerable properties, including customProp!
for (const key in arr) {
  console.log(key);  // '0', '1', '2', 'customProp'
}

// for...of only iterates array values
for (const value of arr) {
  console.log(value);  // 'a', 'b', 'c'
}
```

### Object.keys/values/entries

```javascript
const obj = { a: 1, b: 2, c: 3 };

// Keys
for (const key of Object.keys(obj)) {
  console.log(key);  // 'a', 'b', 'c'
}

// Values
for (const value of Object.values(obj)) {
  console.log(value);  // 1, 2, 3
}

// Entries
for (const [key, value] of Object.entries(obj)) {
  console.log(key, value);  // 'a' 1, 'b' 2, 'c' 3
}
```

## Common Pitfalls

1. **Using `for...in` on arrays**: Iterates indices as strings and includes non-index properties.
2. **Expecting `for...of` on plain objects**: Objects are not iterable by default—use `Object.entries()`.
3. **Mutating during iteration**: Can cause skipped elements or infinite loops.

## Best Practices

- **Use `for...of` for arrays and iterables**: Cleaner and safer than `for...in`.
- **Use `for...in` only for objects**: When you need to enumerate properties.
- **Use `Object.entries()` for object iteration**: Combines key and value access.
- **Consider array methods**: `forEach`, `map`, `filter` are often more expressive.

## Summary

`for...of` iterates over values of iterables (arrays, strings, Maps, Sets). `for...in` iterates over enumerable property keys and should be used for objects, not arrays. Use `Object.entries()` with `for...of` to iterate objects with both keys and values.

## Resources

- [MDN: for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) — for...of loop reference
- [MDN: for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) — for...in loop reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*