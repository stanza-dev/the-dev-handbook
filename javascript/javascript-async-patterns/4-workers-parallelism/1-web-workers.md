---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-web-workers"
---

# Web Workers Basics

## Introduction

JavaScript is single-threaded, but Web Workers enable true parallelism. A worker runs scripts in a background thread, freeing the main thread for UI responsiveness. Heavy computations, image processing, and data parsing—all can move off the main thread.

## Key Concepts

**Web Worker**: A script running in a background thread, separate from the main thread.

**postMessage()**: Method to send data between main thread and worker.

**Dedicated vs Shared Workers**: Dedicated workers belong to one script; shared workers can be accessed by multiple.

## Real World Context

Syntax highlighting in editors, WebAssembly compilation, zip/unzip operations, chart rendering—anything CPU-intensive benefits from workers. Libraries like Partytown use workers to run third-party scripts off main thread.

## Deep Dive

### Basic Setup

```javascript
// main.js
const worker = new Worker('worker.js');

// Send data to worker
worker.postMessage({ type: 'process', data: [1, 2, 3] });

// Receive results
worker.onmessage = (event) => {
  console.log('Result:', event.data);
};

// Handle errors
worker.onerror = (error) => {
  console.error('Worker error:', error.message);
};

// Terminate when done
worker.terminate();
```

```javascript
// worker.js
self.onmessage = (event) => {
  const { type, data } = event.data;
  
  if (type === 'process') {
    const result = data.map(n => n * 2);
    self.postMessage(result);
  }
};
```

### What Workers CAN and CAN'T Access

```javascript
// Workers CAN use:
// - setTimeout, setInterval
// - fetch, XMLHttpRequest
// - IndexedDB
// - WebSockets
// - importScripts() for loading scripts
// - Crypto API
// - navigator (limited)

// Workers CANNOT access:
// - DOM (document, window)
// - Parent objects directly
// - localStorage, sessionStorage
// - alert(), confirm(), prompt()
```

### Inline Workers with Blob

```javascript
const workerCode = `
  self.onmessage = (e) => {
    const result = e.data.map(n => n * 2);
    self.postMessage(result);
  };
`;

const blob = new Blob([workerCode], { type: 'application/javascript' });
const worker = new Worker(URL.createObjectURL(blob));

worker.postMessage([1, 2, 3]);
worker.onmessage = (e) => console.log(e.data); // [2, 4, 6]
```

### Module Workers (ES2019)

```javascript
// Create a module worker
const worker = new Worker('worker.mjs', { type: 'module' });

// worker.mjs can use import/export
import { processData } from './utils.mjs';

self.onmessage = (e) => {
  self.postMessage(processData(e.data));
};
```

## Common Pitfalls

1. **Trying to access DOM**: Workers have no DOM access.
2. **Large data transfer overhead**: Serialization takes time.
3. **Forgetting to terminate**: Leaks memory.

## Best Practices

- **Use Transferable Objects**: For large ArrayBuffers.
- **Terminate when done**: Avoid memory leaks.
- **Use module workers**: Better code organization.
- **Pool workers for repeated tasks**: Reuse instead of create.

## Summary

Web Workers run JavaScript in background threads. Communication uses postMessage(). Workers cannot access DOM but can use fetch, IndexedDB, etc. Use for CPU-intensive tasks. Consider Transferable Objects for large data.

## Resources

- [MDN: Web Workers API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) — Web Workers API reference
- [MDN: Using Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) — Guide to using workers

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*