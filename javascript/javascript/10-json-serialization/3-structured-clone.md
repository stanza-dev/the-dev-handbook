---
source_course: "javascript"
source_lesson: "javascript-structured-clone"
---

# Modern Cloning: structuredClone

## Introduction

For years, developers used `JSON.parse(JSON.stringify(obj))` for deep cloning—a hack with many limitations. `structuredClone()` (ES2022) is the proper solution, supporting more types and handling edge cases correctly.

## Key Concepts

**Deep Clone**: A complete copy where nested objects are also copied.

**Shallow Clone**: Only top-level properties are copied; nested objects are shared.

**Structured Cloning**: Browser algorithm for copying complex values.

## Real World Context

Copying state in Redux/Zustand, duplicating form data, isolating test fixtures—deep cloning is common. structuredClone handles cases JSON can't.

## Deep Dive

### Basic Usage

```javascript
const original = {
  name: 'Alice',
  scores: [95, 87, 92],
  metadata: { created: new Date() }
};

const clone = structuredClone(original);

// Changes don't affect original
clone.scores.push(100);
clone.metadata.created.setFullYear(2000);

original.scores;  // [95, 87, 92] (unchanged)
original.metadata.created.getFullYear();  // Original year
```

### What structuredClone Supports

```javascript
// All these are cloned properly:
const complex = {
  date: new Date(),
  regex: /hello/gi,
  map: new Map([['a', 1]]),
  set: new Set([1, 2, 3]),
  arrayBuffer: new ArrayBuffer(8),
  typedArray: new Uint8Array([1, 2, 3]),
  blob: new Blob(['hello']),
  error: new Error('oops')
};

const cloned = structuredClone(complex);
cloned.date instanceof Date;  // true!
cloned.map instanceof Map;    // true!
```

### What structuredClone Cannot Clone

```javascript
// Functions - throw error
structuredClone({ fn: () => {} });
// DOMException: function cannot be cloned

// Symbols - throw error
structuredClone({ sym: Symbol('id') });
// DOMException

// DOM nodes - throw error
structuredClone({ el: document.body });
// DOMException

// Property descriptors/getters/setters - lost
const obj = {
  get value() { return 42; }
};
structuredClone(obj);  // { value: 42 } - getter lost

// Prototype chain - lost
class User { greet() {} }
const user = new User();
const clone = structuredClone(user);
clone instanceof User;  // false!
clone.greet;            // undefined
```

### Comparison Table

```javascript
// Feature comparison:
// 
// Feature          | JSON hack | structuredClone
// -----------------|-----------|----------------
// Dates            | Strings   | Date objects
// undefined        | Lost      | Preserved
// Map/Set          | Lost      | Cloned
// Circular refs    | Error     | Handled!
// Functions        | Lost      | Error
// RegExp           | Empty {}  | Cloned
// Prototypes       | Lost      | Lost
// Speed            | Slow      | Faster
```

### Circular References

```javascript
// JSON fails with circular references
const obj = { name: 'Alice' };
obj.self = obj;
// JSON.stringify(obj);  // TypeError

// structuredClone handles them!
const clone = structuredClone(obj);
clone.self === clone;  // true (circular reference preserved)
```

### Transfer Option

```javascript
// Transfer ownership instead of copying (for performance)
const buffer = new ArrayBuffer(1024);
const clone = structuredClone({ data: buffer }, {
  transfer: [buffer]
});

buffer.byteLength;  // 0 (transferred, no longer usable)
clone.data.byteLength;  // 1024
```

## Common Pitfalls

1. **Expecting functions to clone**: They throw; filter them first.
2. **Expecting prototype preservation**: structuredClone creates plain objects.
3. **Using in older environments**: Check browser support or use polyfill.

## Best Practices

- **Use structuredClone for deep cloning**: Not JSON hack.
- **Filter non-cloneable values first**: If object might have functions.
- **Use transfer for large buffers**: Avoids copying overhead.
- **Check class instances**: Prototype is lost; reconstruct if needed.

## Summary

`structuredClone()` is the modern way to deep clone objects. It handles Dates, Maps, Sets, circular references, and more. Functions, symbols, and DOM nodes cannot be cloned. Prototypes are not preserved. Use transfer option for large ArrayBuffers.

## Resources

- [MDN: structuredClone()](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone) — structuredClone reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*