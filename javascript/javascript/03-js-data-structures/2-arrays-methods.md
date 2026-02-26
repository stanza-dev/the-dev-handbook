---
source_course: "javascript"
source_lesson: "javascript-arrays-methods"
---

# Array Transformation Methods

## Introduction

JavaScript arrays come with powerful methods for transforming data. `map()`, `filter()`, and `reduce()` are the cornerstone of functional data processing. These methods are chainable, immutable, and form the foundation of modern JavaScript programming.

## Key Concepts

**Transformation Method**: A method that creates a new array/value without modifying the original.

**Callback Function**: A function passed to the method that defines how each element is processed.

**Method Chaining**: Calling multiple methods in sequence on the result of the previous method.

## Real World Context

Filtering products by category, transforming API data for display, calculating totals, grouping items—these operations are everywhere in real applications. Understanding these methods is essential for React, data processing, and any modern JavaScript work.

## Deep Dive

### map() - Transform Each Element

```javascript
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(n => n * 2);
// [2, 4, 6, 8]

// With index
const indexed = numbers.map((n, i) => `${i}: ${n}`);
// ['0: 1', '1: 2', '2: 3', '3: 4']

// Transform objects
const users = [{ name: 'Alice' }, { name: 'Bob' }];
const names = users.map(user => user.name);
// ['Alice', 'Bob']
```

### filter() - Select Elements

```javascript
const numbers = [1, 2, 3, 4, 5, 6];
const evens = numbers.filter(n => n % 2 === 0);
// [2, 4, 6]

// Filter objects
const products = [
  { name: 'A', price: 10 },
  { name: 'B', price: 25 },
  { name: 'C', price: 5 }
];
const affordable = products.filter(p => p.price < 20);
// [{ name: 'A', price: 10 }, { name: 'C', price: 5 }]
```

### reduce() - Accumulate to Single Value

```javascript
const numbers = [1, 2, 3, 4];

// Sum
const sum = numbers.reduce((acc, n) => acc + n, 0);
// 10

// Max value
const max = numbers.reduce((acc, n) => n > acc ? n : acc, -Infinity);
// 4

// Group by property
const items = [
  { type: 'fruit', name: 'apple' },
  { type: 'veg', name: 'carrot' },
  { type: 'fruit', name: 'banana' }
];
const grouped = items.reduce((acc, item) => {
  (acc[item.type] ??= []).push(item.name);
  return acc;
}, {});
// { fruit: ['apple', 'banana'], veg: ['carrot'] }
```

### Method Chaining

```javascript
const products = [
  { name: 'A', price: 10, inStock: true },
  { name: 'B', price: 25, inStock: false },
  { name: 'C', price: 15, inStock: true }
];

const result = products
  .filter(p => p.inStock)
  .map(p => p.price)
  .reduce((sum, price) => sum + price, 0);
// 25
```

### find() and findIndex()

```javascript
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

const user = users.find(u => u.id === 2);
// { id: 2, name: 'Bob' }

const index = users.findIndex(u => u.id === 2);
// 1
```

### Object.groupBy() (ES2024+)

The `reduce()` grouping pattern shown above now has a built-in alternative:

```javascript
const items = [
  { type: 'fruit', name: 'apple' },
  { type: 'veg', name: 'carrot' },
  { type: 'fruit', name: 'banana' }
];

// Object.groupBy — returns a null-prototype object
const grouped = Object.groupBy(items, item => item.type);
// { fruit: [{...}, {...}], veg: [{...}] }

// Map.groupBy — returns a Map (useful for non-string keys)
const byLength = Map.groupBy(items, item => item.name.length);
// Map { 5 => [{apple}], 6 => [{carrot}, {banana}] }
```

`Object.groupBy()` replaces the common reduce-based pattern with a single readable call. Use `Map.groupBy()` when your grouping keys are objects, numbers, or other non-string values.

## Common Pitfalls

1. **Forgetting `return` in reduce**: The accumulator won't update.
2. **Using `map()` for side effects**: Use `forEach()` instead if you don't need a new array.
3. **Missing initial value in reduce**: Can cause errors on empty arrays.

## Best Practices

- **Always provide initial value to reduce()**: Prevents errors and clarifies intent.
- **Use `find()` for single element**: Not `filter()[0]`.
- **Keep callbacks pure**: Don't mutate external state.
- **Chain methods for readability**: But break into variables for debugging.

## Summary

`map()` transforms each element, `filter()` selects elements, `reduce()` accumulates to a single value. These methods are immutable, chainable, and fundamental to functional JavaScript. Use `find()` for single elements and always provide initial values to `reduce()`.

## Code Examples

**map() - Transform Each Element**

```javascript
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(n => n * 2);
// [2, 4, 6, 8]

// With index
const indexed = numbers.map((n, i) => `${i}: ${n}`);
// ['0: 1', '1: 2', '2: 3', '3: 4']

// Transform objects
const users = [{ name: 'Alice' }, { name: 'Bob' }];
const names = users.map(user => user.name);
// ['Alice', 'Bob']
```

**filter() - Select Elements**

```javascript
const numbers = [1, 2, 3, 4, 5, 6];
const evens = numbers.filter(n => n % 2 === 0);
// [2, 4, 6]

// Filter objects
const products = [
  { name: 'A', price: 10 },
  { name: 'B', price: 25 },
  { name: 'C', price: 5 }
];
const affordable = products.filter(p => p.price < 20);
// [{ name: 'A', price: 10 }, { name: 'C', price: 5 }]
```


## Resources

- [MDN: Array.prototype.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) — map() method reference
- [MDN: Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) — reduce() method reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*