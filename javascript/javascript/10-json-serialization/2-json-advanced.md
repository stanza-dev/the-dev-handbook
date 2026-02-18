---
source_course: "javascript"
source_lesson: "javascript-json-advanced"
---

# Advanced JSON Techniques

## Introduction

Beyond basic stringify/parse, JSON methods accept powerful transformer functions. These let you control exactly how data is serialized and reconstructed—essential for handling dates, filtering sensitive data, and custom object types.

## Key Concepts

**Replacer**: Function or array controlling what's included in stringify output.

**Reviver**: Function transforming values during parse.

**toJSON()**: Method objects can implement for custom serialization.

## Deep Dive

### The Replacer Parameter

```javascript
const user = {
  name: 'Alice',
  password: 'secret123',
  email: 'alice@example.com',
  age: 30
};

// Array replacer - whitelist properties
JSON.stringify(user, ['name', 'email']);
// '{"name":"Alice","email":"alice@example.com"}'

// Function replacer - transform values
JSON.stringify(user, (key, value) => {
  if (key === 'password') return undefined;  // Skip
  if (typeof value === 'string') return value.toUpperCase();
  return value;
});
// '{"name":"ALICE","email":"ALICE@EXAMPLE.COM","age":30}'
```

### The Reviver Parameter

```javascript
// Problem: Dates become strings
const json = '{"created":"2024-12-25T00:00:00.000Z"}';
const obj1 = JSON.parse(json);
typeof obj1.created;  // 'string' - not a Date!

// Solution: Use reviver
const obj2 = JSON.parse(json, (key, value) => {
  if (key === 'created') return new Date(value);
  return value;
});
obj2.created instanceof Date;  // true!

// Generic ISO date reviver
const isoDatePattern = /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}/;
function dateReviver(key, value) {
  if (typeof value === 'string' && isoDatePattern.test(value)) {
    return new Date(value);
  }
  return value;
}

JSON.parse(json, dateReviver);
```

### Custom toJSON() Method

```javascript
class User {
  constructor(name, password) {
    this.name = name;
    this.password = password;  // Sensitive!
  }
  
  // Controls JSON representation
  toJSON() {
    return {
      name: this.name,
      type: 'User'
      // password NOT included
    };
  }
}

const user = new User('Alice', 'secret');
JSON.stringify(user);  // '{"name":"Alice","type":"User"}'

// Date's built-in toJSON
const date = new Date('2024-12-25');
date.toJSON();  // '2024-12-25T00:00:00.000Z'
```

### Deep Clone with JSON

```javascript
// Quick deep clone (with limitations)
const original = { a: 1, b: { c: 2 } };
const clone = JSON.parse(JSON.stringify(original));

clone.b.c = 999;
original.b.c;  // Still 2!

// Limitations:
// - Loses undefined, functions, symbols
// - Dates become strings
// - Doesn't handle circular references
// - Slower than structuredClone()

// Modern alternative (ES2022+)
const betterClone = structuredClone(original);
```

### Handling BigInt

```javascript
// BigInt throws by default
const big = { value: 12345678901234567890n };
// JSON.stringify(big);  // TypeError: BigInt can't be serialized

// Solution: custom toJSON or replacer
JSON.stringify(big, (key, value) => {
  if (typeof value === 'bigint') return value.toString();
  return value;
});
// '{"value":"12345678901234567890"}'
```

## Common Pitfalls

1. **Losing Date types**: Always use a reviver for dates.
2. **toJSON must return JSON-compatible value**: Not a string!
3. **Replacer function called for root too**: Key is empty string.

## Best Practices

- **Implement toJSON for custom classes**: Control serialization.
- **Use reviver for typed parsing**: Reconstruct dates, custom types.
- **Use structuredClone() over JSON for cloning**: Handles more types.
- **Filter sensitive data in toJSON**: Don't serialize passwords.

## Summary

Replacer filters/transforms during stringify. Reviver transforms during parse. Classes can implement `toJSON()` for custom serialization. Use revivers to restore dates and custom types. Prefer `structuredClone()` for deep cloning in modern code.

## Resources

- [MDN: JSON.parse() reviver](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse#using_the_reviver_parameter) — Reviver parameter usage
- [MDN: structuredClone()](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone) — Modern deep cloning

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*