---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-transferable-objects"
---

# Transferable Objects & SharedArrayBuffer

## Introduction

postMessage() copies data by default—slow for large datasets. Transferable objects like ArrayBuffer can be transferred (zero-copy), and SharedArrayBuffer enables true shared memory between threads.

## Key Concepts

**Transferable Object**: Data that can be transferred (not copied) between threads.

**SharedArrayBuffer**: Memory shared between main thread and workers.

**Atomics**: API for thread-safe operations on shared memory.

## Deep Dive

### Transferable Objects

```javascript
// Without transfer - data is COPIED (slow for large data)
const buffer = new ArrayBuffer(100_000_000);  // 100MB
worker.postMessage(buffer);  // Copied! 200MB total now

// With transfer - data is MOVED (fast, zero-copy)
worker.postMessage(buffer, [buffer]);
buffer.byteLength;  // 0 - it's been transferred!
```

### SharedArrayBuffer

```javascript
// main.js
const sharedBuffer = new SharedArrayBuffer(1024);
const sharedArray = new Int32Array(sharedBuffer);

worker.postMessage({ buffer: sharedBuffer });

// Both threads can read/write sharedArray
sharedArray[0] = 42;

// worker.js
self.onmessage = (e) => {
  const sharedArray = new Int32Array(e.data.buffer);
  console.log(sharedArray[0]);  // 42
  sharedArray[0] = 100;  // Main thread sees this!
};
```

### Atomics for Thread Safety

```javascript
const sharedArray = new Int32Array(sharedBuffer);

// Atomic operations prevent race conditions
Atomics.add(sharedArray, 0, 5);      // Atomic increment
Atomics.load(sharedArray, 0);        // Atomic read
Atomics.store(sharedArray, 0, 10);   // Atomic write
Atomics.compareExchange(sharedArray, 0, 10, 20); // CAS

// Wait/notify for coordination
Atomics.wait(sharedArray, 0, expectedValue);  // Block until changed
Atomics.notify(sharedArray, 0, 1);  // Wake one waiting thread
```

### Security Requirements

```javascript
// SharedArrayBuffer requires these headers:
// Cross-Origin-Opener-Policy: same-origin
// Cross-Origin-Embedder-Policy: require-corp

// Check if available
if (typeof SharedArrayBuffer !== 'undefined') {
  // Can use SharedArrayBuffer
}
```

### Atomics.waitAsync() (ES2024+)

`Atomics.wait()` blocks the calling thread — unusable on the main thread. `Atomics.waitAsync()` returns a promise instead:

```javascript
// Main thread — non-blocking wait
const sharedArray = new Int32Array(sharedBuffer);
const result = Atomics.waitAsync(sharedArray, 0, 0);

if (result.async) {
  result.value.then(() => {
    console.log('Worker updated the value!');
    console.log('New value:', Atomics.load(sharedArray, 0));
  });
} else {
  console.log('Already changed:', result.value);
}

// Worker thread — notify main thread
Atomics.store(sharedArray, 0, 42);
Atomics.notify(sharedArray, 0);
```

`Atomics.waitAsync()` enables main-thread coordination with workers without blocking the event loop. The returned object has `{ async: true, value: Promise }` for async waits or `{ async: false, value: 'not-equal'|'timed-out' }` for immediate results.

## Common Pitfalls

1. **Using transferred buffer after transfer**: It's empty!
2. **Race conditions with SharedArrayBuffer**: Use Atomics.
3. **Missing security headers**: SharedArrayBuffer won't be available.

## Summary

Transferable objects enable zero-copy data transfer. SharedArrayBuffer provides true shared memory. Use Atomics for thread-safe operations. Ensure proper security headers for SharedArrayBuffer.

## Code Examples

**Transferable Objects**

```javascript
// Without transfer - data is COPIED (slow for large data)
const buffer = new ArrayBuffer(100_000_000);  // 100MB
worker.postMessage(buffer);  // Copied! 200MB total now

// With transfer - data is MOVED (fast, zero-copy)
worker.postMessage(buffer, [buffer]);
buffer.byteLength;  // 0 - it's been transferred!
```

**SharedArrayBuffer**

```javascript
// main.js
const sharedBuffer = new SharedArrayBuffer(1024);
const sharedArray = new Int32Array(sharedBuffer);

worker.postMessage({ buffer: sharedBuffer });

// Both threads can read/write sharedArray
sharedArray[0] = 42;

// worker.js
self.onmessage = (e) => {
  const sharedArray = new Int32Array(e.data.buffer);
  console.log(sharedArray[0]);  // 42
  sharedArray[0] = 100;  // Main thread sees this!
};
```


## Resources

- [MDN: Transferable objects](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Transferable_objects) — Transferable objects guide
- [MDN: SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer) — SharedArrayBuffer reference

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*