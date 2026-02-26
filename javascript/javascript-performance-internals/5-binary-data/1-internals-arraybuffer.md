---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-arraybuffer"
---

## Introduction

JavaScript was originally designed for manipulating text and DOM elements, but modern web applications frequently need to work with raw binary data. Whether you're processing images, handling WebSocket streams, or interfacing with WebGL, you need a way to read and write bytes directly. **ArrayBuffer** and **TypedArrays** provide this low-level binary data interface, bridging the gap between JavaScript's high-level abstractions and the raw memory that hardware understands.

## Key Concepts

- **ArrayBuffer**: A fixed-length, contiguous block of raw binary memory. It cannot be read or written directly — you must use a view.
- **TypedArray**: A family of array-like objects that provide a typed view over an ArrayBuffer (e.g., `Uint8Array`, `Int32Array`, `Float64Array`).
- **SharedArrayBuffer**: A variant of ArrayBuffer that can be shared across Web Workers, enabling true shared-memory concurrency with `Atomics`.
- **Uint8Array**: A TypedArray where each element is an unsigned 8-bit integer (0–255). The most commonly used typed array.
- **Float64Array**: A TypedArray where each element is a 64-bit IEEE 754 floating-point number, identical to a regular JavaScript `Number`.

## Real World Context

Binary data manipulation is essential across many domains of modern web development. **WebGL** requires vertex data packed into `Float32Array` buffers for GPU rendering. **Web Audio** processes sound samples as `Float32Array` data. **File processing** — reading images, parsing PDFs, or handling ZIP archives — demands byte-level access through typed arrays. **WebSocket binary frames** transmit protocol buffers and other compact binary formats using `ArrayBuffer`. Understanding these primitives unlocks high-performance data handling that string-based approaches simply cannot match.

## Deep Dive

An `ArrayBuffer` allocates a fixed block of memory. You specify the size in bytes at creation time, and it is zero-initialized:

```javascript
const buffer = new ArrayBuffer(16); // 16 bytes of zeroed memory
console.log(buffer.byteLength); // 16
```

To read or write data, you create one or more **typed array views** over the buffer:

```javascript
const uint8 = new Uint8Array(buffer);    // 16 elements (1 byte each)
const int32 = new Int32Array(buffer);     // 4 elements (4 bytes each)
const float64 = new Float64Array(buffer); // 2 elements (8 bytes each)
```

All three views reference the **same underlying memory**. Writing through one view is immediately visible through the others, since they are just different lenses on the same bytes. This is powerful but requires care — overlapping writes can produce unexpected values when interpreted through a different type.

The full set of TypedArray types covers every common numeric width:

| Type | Bytes | Range |
|------|-------|-------|
| `Int8Array` | 1 | -128 to 127 |
| `Uint8Array` | 1 | 0 to 255 |
| `Uint8ClampedArray` | 1 | 0 to 255 (clamped) |
| `Int16Array` | 2 | -32768 to 32767 |
| `Uint16Array` | 2 | 0 to 65535 |
| `Int32Array` | 4 | -2^31 to 2^31-1 |
| `Uint32Array` | 4 | 0 to 2^32-1 |
| `Float32Array` | 4 | IEEE 754 single |
| `Float64Array` | 8 | IEEE 754 double |
| `BigInt64Array` | 8 | -2^63 to 2^63-1 |
| `BigUint64Array` | 8 | 0 to 2^64-1 |

**ES2025** introduces **Float16Array**, which enables half-precision (16-bit) float storage. This is particularly valuable for ML inference and GPU data pipelines where memory bandwidth matters more than precision. Alongside it, `Math.f16round()` rounds a number to half-precision float representation:

```javascript
const f16 = new Float16Array(4);
f16[0] = 1.5;
console.log(f16[0]); // 1.5
console.log(Math.f16round(1.337)); // Rounds to nearest float16
```

## Common Pitfalls

- **Alignment errors**: Creating a `Float64Array` from a buffer at an offset that is not a multiple of 8 throws a `RangeError`. Always ensure your byte offset is aligned to the element size of the typed array.
- **Overflow wrapping vs. clamping**: `Uint8Array` wraps on overflow (256 becomes 0), while `Uint8ClampedArray` clamps (256 becomes 255). Choosing the wrong one produces silent data corruption.
- **Detached buffers**: Transferring an `ArrayBuffer` to a Worker via `postMessage` detaches it, making all existing views throw on access. Always check that you no longer need local access before transferring.

## Best Practices

- **Prefer `Uint8Array` as a general-purpose byte container.** It is the most widely supported type across Web APIs (fetch, streams, WebSocket, file readers).
- **Create typed arrays with their own allocation when you don't need a shared buffer.** Writing `new Float32Array(100)` is simpler and less error-prone than manually creating an `ArrayBuffer` and wrapping it.
- **Use `TypedArray.prototype.slice()` to create independent copies** when you need to prevent mutations from propagating through shared buffer references.

## Summary

`ArrayBuffer` is the foundational primitive for binary data in JavaScript — a fixed block of raw bytes. TypedArrays provide typed, array-like views for reading and writing numeric values at specific byte widths. Multiple views can share the same buffer, enabling efficient reinterpretation of data. ES2025 adds `Float16Array` for half-precision floats, expanding the ecosystem for ML and GPU workloads. Mastering these primitives is the first step to high-performance binary data processing in JavaScript.

## Code Examples

**ArrayBuffer with typed views and ES2025 Float16Array**

```javascript
// Create a buffer and typed views
const buffer = new ArrayBuffer(16); // 16 bytes
const uint8 = new Uint8Array(buffer); // 16 elements
const float64 = new Float64Array(buffer); // 2 elements

uint8[0] = 255;
console.log(float64[0]); // Interprets same bytes as float64

// ES2025: Float16Array for half-precision
const f16 = new Float16Array(4);
f16[0] = 1.5;
console.log(f16[0]); // 1.5 (half-precision)
console.log(Math.f16round(1.337)); // Rounds to float16
```


## Resources

- [MDN: ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) — ArrayBuffer for raw binary data storage
- [MDN: TypedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) — TypedArray views for binary data access

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*