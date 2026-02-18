---
source_course: "javascript"
source_lesson: "javascript-maps-and-sets"
---

# Maps and Sets

## Introduction

While objects and arrays handle most data storage needs, ES6 introduced Map and Set for specialized use cases. Maps allow any value as a key (not just strings), and Sets store only unique values. These data structures solve problems that are awkward with plain objects and arrays.

## Key Concepts

**Map**: A collection of key-value pairs where keys can be any type (objects, functions, primitives).

**Set**: A collection of unique values—duplicates are automatically ignored.

**WeakMap/WeakSet**: Variants that hold weak references, allowing garbage collection.

## Real World Context

Maps are perfect for caching (using objects as keys), counting occurrences, and storing metadata about DOM elements. Sets excel at deduplication, tracking unique visitors, and implementing tags or categories.

## Deep Dive

### Map Basics

```javascript
const map = new Map();

// Set and get
map.set('name', 'Alice');
map.set(1, 'one');           // Number as key
map.set({ id: 1 }, 'user');  // Object as key

map.get('name');  // 'Alice'
map.get(1);       // 'one'

// Size and existence
map.size;           // 3
map.has('name');    // true

// Delete
map.delete('name');
map.clear();  // Remove all
```

### Map Iteration

```javascript
const map = new Map([
  ['a', 1],
  ['b', 2],
  ['c', 3]
]);

// Iterate entries
for (const [key, value] of map) {
  console.log(key, value);
}

// Other iterators
map.keys();    // Iterator of keys
map.values();  // Iterator of values
map.entries(); // Iterator of [key, value]

// forEach
map.forEach((value, key) => console.log(key, value));
```

### Map vs Object

```javascript
// Objects convert keys to strings
const obj = {};
obj[1] = 'one';
obj['1'] = 'string one';
console.log(obj);  // { '1': 'string one' } - key collision!

// Maps preserve key types
const map = new Map();
map.set(1, 'number one');
map.set('1', 'string one');
map.get(1);   // 'number one'
map.get('1'); // 'string one'  - no collision!
```

### Set Basics

```javascript
const set = new Set([1, 2, 2, 3, 3, 3]);
console.log(set.size);  // 3 (duplicates removed)

// Add and check
set.add(4);
set.has(2);  // true

// Delete
set.delete(1);
set.clear();  // Remove all

// Convert to array
const arr = [...set];
const arr2 = Array.from(set);
```

### Set for Deduplication

```javascript
const numbers = [1, 2, 2, 3, 3, 3, 4];
const unique = [...new Set(numbers)];
// [1, 2, 3, 4]

// Set operations
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);

// Union
const union = new Set([...a, ...b]);  // {1, 2, 3, 4}

// Intersection
const intersection = new Set([...a].filter(x => b.has(x)));  // {2, 3}

// Difference
const difference = new Set([...a].filter(x => !b.has(x)));  // {1}
```

### WeakMap and WeakSet

```javascript
// Keys must be objects, and are weakly held
const weakMap = new WeakMap();
let obj = { id: 1 };
weakMap.set(obj, 'metadata');

obj = null;  // Now the entry can be garbage collected

// Use case: storing private data for objects
const privateData = new WeakMap();
class User {
  constructor(name) {
    privateData.set(this, { name });
  }
  getName() {
    return privateData.get(this).name;
  }
}
```

## Common Pitfalls

1. **Using objects as Map keys and expecting equality**: `map.get({})` won't find `map.set({}, value)` (different object references).
2. **Forgetting Map preserves insertion order**: Unlike plain objects in older engines.
3. **Expecting Set methods like filter/map**: Sets don't have these; convert to array first.

## Best Practices

- **Use Map when keys aren't strings**: Or when key type matters.
- **Use Set for uniqueness**: Instead of array + includes() checks.
- **Use WeakMap for object metadata**: Allows garbage collection.
- **Convert Set to Array for transformations**: `[...set].map(...)`.

## Summary

Maps store key-value pairs with any key type and maintain insertion order. Sets store unique values and are perfect for deduplication. Both are iterable and have `.size` property. Use WeakMap/WeakSet when you don't want to prevent garbage collection of keys.

## Resources

- [MDN: Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) — Map reference
- [MDN: Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) — Set reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*