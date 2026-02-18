---
source_course: "javascript"
source_lesson: "javascript-json-basics"
---

# JSON Fundamentals

## Introduction

JSON (JavaScript Object Notation) is the universal language of data exchange on the web. APIs send JSON, configuration files use JSON, and localStorage stores JSON strings. Mastering JSON serialization and parsing is essential for any JavaScript developer.

## Key Concepts

**JSON**: A text format for structured data, based on JavaScript object syntax.

**Serialization**: Converting JavaScript values to JSON strings (`stringify`).

**Parsing**: Converting JSON strings to JavaScript values (`parse`).

## Real World Context

Every API call involves JSON. localStorage only stores strings, so you stringify/parse objects. Configuration files, data export/import, WebSocket messages—JSON is everywhere.

## Deep Dive

### JSON Syntax Rules

```javascript
// Valid JSON
{
  "name": "Alice",          // Keys MUST be double-quoted
  "age": 30,                // Numbers
  "active": true,           // Booleans
  "address": null,          // null
  "tags": ["admin", "user"], // Arrays
  "profile": {              // Nested objects
    "bio": "Developer"
  }
}

// NOT valid in JSON (but valid in JS):
// - Single quotes: 'name'
// - Unquoted keys: { name: "Alice" }
// - Trailing commas: { "a": 1, }
// - Comments: // or /* */
// - undefined, functions, symbols
```

### JSON.stringify()

```javascript
const user = { name: 'Alice', age: 30 };

// Basic usage
JSON.stringify(user);  // '{"name":"Alice","age":30}'

// Pretty printing (2 space indent)
JSON.stringify(user, null, 2);
// {
//   "name": "Alice",
//   "age": 30
// }

// Tab indent
JSON.stringify(user, null, '\t');
```

### JSON.parse()

```javascript
const jsonStr = '{"name":"Alice","age":30}';

// Basic usage
const user = JSON.parse(jsonStr);
user.name;  // 'Alice'

// Invalid JSON throws
try {
  JSON.parse('invalid');  // SyntaxError
} catch (e) {
  console.error('Invalid JSON');
}

// Safe parsing pattern
function safeParse(json, fallback = null) {
  try {
    return JSON.parse(json);
  } catch {
    return fallback;
  }
}
```

### What JSON.stringify Skips

```javascript
const obj = {
  name: 'Alice',
  age: undefined,        // Skipped!
  greet: function() {},  // Skipped!
  id: Symbol('id'),      // Skipped!
};

JSON.stringify(obj);  // '{"name":"Alice"}'

// In arrays, these become null:
JSON.stringify([1, undefined, 3]);  // '[1,null,3]'
```

### Circular Reference Error

```javascript
const obj = { name: 'Alice' };
obj.self = obj;  // Circular reference

// JSON.stringify(obj);  // TypeError: Converting circular structure
```

## Common Pitfalls

1. **Date objects become strings**: `new Date()` stringifies to ISO string.
2. **undefined is skipped**: Properties with undefined disappear.
3. **No error for invalid JSON until parse()**: Always validate.

## Best Practices

- **Always wrap parse in try/catch**: Invalid JSON throws.
- **Use pretty printing for debugging**: `JSON.stringify(obj, null, 2)`.
- **Check for circular references**: Use libraries for complex objects.
- **Validate JSON structure**: Don't assume parsed data has expected shape.

## Summary

JSON is text-based data format with strict syntax (double quotes, no trailing commas). `JSON.stringify()` converts to string, `JSON.parse()` converts back. undefined, functions, and symbols are skipped. Circular references throw errors. Always wrap `parse()` in try/catch.

## Resources

- [MDN: JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON) — JSON object reference
- [MDN: JSON.stringify()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) — stringify() reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*