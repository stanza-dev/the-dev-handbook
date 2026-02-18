---
source_course: "javascript"
source_lesson: "javascript-objects-fundamentals"
---

# Objects: Properties & Methods

## Introduction

Objects are collections of key-value pairs—the fundamental way to structure and group related data in JavaScript. From simple data containers to complex class instances, objects are everywhere. Understanding object manipulation is core to JavaScript mastery.

## Key Concepts

**Object**: An unordered collection of properties (key-value pairs).

**Property**: A key-value pair where the key is a string (or Symbol) and the value can be anything.

**Method**: A property whose value is a function.

## Real World Context

User profiles, API responses, configuration objects, component state—nearly all structured data in JavaScript applications is represented as objects. JSON (JavaScript Object Notation) is the standard data interchange format.

## Deep Dive

### Creating Objects

```javascript
// Object literal
const user = {
  name: 'Alice',
  age: 30,
  email: 'alice@example.com'
};

// Shorthand property names
const name = 'Bob';
const age = 25;
const bob = { name, age };  // { name: 'Bob', age: 25 }

// Computed property names
const key = 'dynamicKey';
const obj = { [key]: 'value' };  // { dynamicKey: 'value' }
```

### Accessing Properties

```javascript
const user = { name: 'Alice', 'full-name': 'Alice Smith' };

// Dot notation
user.name;  // 'Alice'

// Bracket notation (required for special keys)
user['full-name'];  // 'Alice Smith'
user['name'];       // 'Alice'

// Dynamic access
const key = 'name';
user[key];  // 'Alice'

// Optional chaining
user?.address?.city;  // undefined (no error)
```

### Modifying Objects

```javascript
const user = { name: 'Alice' };

// Add/update properties
user.age = 30;
user.name = 'Alicia';

// Delete properties
delete user.age;

// Check property existence
'name' in user;  // true
user.hasOwnProperty('name');  // true
Object.hasOwn(user, 'name');  // true (ES2022, preferred)
```

### Object Methods

```javascript
const user = { name: 'Alice', age: 30 };

// Get keys, values, entries
Object.keys(user);     // ['name', 'age']
Object.values(user);   // ['Alice', 30]
Object.entries(user);  // [['name', 'Alice'], ['age', 30]]

// Create from entries
Object.fromEntries([['a', 1], ['b', 2]]);  // { a: 1, b: 2 }

// Merge objects
const defaults = { theme: 'light', lang: 'en' };
const prefs = { theme: 'dark' };
const merged = { ...defaults, ...prefs };  // { theme: 'dark', lang: 'en' }

// Object.assign (mutates first arg)
Object.assign({}, defaults, prefs);  // Same result
```

### Destructuring

```javascript
const user = { name: 'Alice', age: 30, city: 'NYC' };

// Basic destructuring
const { name, age } = user;

// Renaming
const { name: userName } = user;  // userName = 'Alice'

// Default values
const { country = 'USA' } = user;  // country = 'USA'

// Rest properties
const { name: n, ...rest } = user;  // rest = { age: 30, city: 'NYC' }
```

## Common Pitfalls

1. **Mutating objects passed to functions**: Objects are passed by reference.
2. **Using `for...in` without `hasOwnProperty`**: May iterate inherited properties.
3. **Confusing shallow vs deep copy**: Spread only copies one level deep.

## Best Practices

- **Use object spread for immutable updates**: `{ ...obj, newProp: value }`.
- **Use `Object.hasOwn()` over `hasOwnProperty()`**: Safer for objects without prototype.
- **Destructure in function parameters**: `function({ name, age }) {}`.
- **Freeze objects for constants**: `Object.freeze(obj)` prevents modification.

## Summary

Objects store key-value pairs accessed via dot or bracket notation. Use `Object.keys/values/entries()` for iteration. Spread syntax (`...`) merges and copies objects. Destructuring extracts properties into variables. Remember objects are passed by reference.

## Resources

- [MDN: Working with Objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects) — Comprehensive guide to JavaScript objects
- [MDN: Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) — Object reference with all static methods

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*