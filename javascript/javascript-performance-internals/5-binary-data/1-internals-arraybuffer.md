---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-arraybuffer"
---

# ArrayBuffer Fundamentals

## Introduction

ArrayBuffer represents raw binary data—a fixed-length block of memory accessed through typed array views.

## Deep Dive

### Creating Buffers

```javascript
const buffer = new ArrayBuffer(16);  // 16 bytes
buffer.byteLength;  // 16
```

### TypedArray Views

```javascript
const buffer = new ArrayBuffer(16);

// Different views of same memory
const uint8 = new Uint8Array(buffer);   // 16 elements
const uint16 = new Uint16Array(buffer); // 8 elements
const int32 = new Int32Array(buffer);   // 4 elements
const float64 = new Float64Array(buffer); // 2 elements

// Views share memory!
int32[0] = 1;
uint8[0];  // Same bytes, different view
```

### Available Types

```javascript
Int8Array    // -128 to 127
Uint8Array   // 0 to 255
Int16Array   // -32768 to 32767
Uint16Array  // 0 to 65535
Int32Array   // -2^31 to 2^31-1
Uint32Array  // 0 to 2^32-1
Float32Array // Single precision
Float64Array // Double precision
```

## Summary

ArrayBuffer holds raw bytes. TypedArrays provide typed views. Multiple views share same buffer.

## Resources

- [MDN: ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) — ArrayBuffer reference

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*