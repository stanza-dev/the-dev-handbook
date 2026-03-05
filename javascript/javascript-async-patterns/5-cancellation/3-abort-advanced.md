---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-abort-advanced"
---

# Advanced Cancellation

## Introduction

AbortController works beyond fetch—with EventListeners, streams, and custom async operations. Understanding these advanced uses enables comprehensive cancellation strategies.

## Deep Dive

### Event Listeners with Signal

```javascript
const controller = new AbortController();

window.addEventListener('resize', handleResize, {
  signal: controller.signal
});

window.addEventListener('scroll', handleScroll, {
  signal: controller.signal  
});

// Remove ALL listeners with one call
controller.abort();
// No need for removeEventListener!
```

### Stream Cancellation

```javascript
const controller = new AbortController();

async function processStream(url) {
  const response = await fetch(url, { signal: controller.signal });
  const reader = response.body.getReader();
  
  try {
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      processChunk(value);
    }
  } finally {
    reader.releaseLock();
  }
}

// Cancel mid-stream
controller.abort();
```

### Promise.withResolvers + AbortController

```javascript
// ES2024: Create controllable promise
function cancellableOperation(signal) {
  const { promise, resolve, reject } = Promise.withResolvers();
  
  signal?.throwIfAborted();  // Fail immediately if already aborted
  
  signal?.addEventListener('abort', () => {
    reject(new DOMException('Aborted', 'AbortError'));
  });
  
  // Async work that eventually calls resolve/reject
  doAsyncWork().then(resolve).catch(reject);
  
  return promise;
}
```

### Batch Operations

```javascript
class BatchProcessor {
  #controller = new AbortController();
  
  async processBatch(items) {
    const results = await Promise.allSettled(
      items.map(item => this.processItem(item))
    );
    return results;
  }
  
  async processItem(item) {
    this.#controller.signal.throwIfAborted();
    return await fetch(`/api/process/${item}`, {
      signal: this.#controller.signal
    });
  }
  
  cancel() {
    this.#controller.abort();
  }
}
```

### Graceful Shutdown Pattern

```javascript
class ApiClient {
  #shutdownController = new AbortController();
  
  async fetch(url, options = {}) {
    const signal = options.signal 
      ? AbortSignal.any([options.signal, this.#shutdownController.signal])
      : this.#shutdownController.signal;
    
    return fetch(url, { ...options, signal });
  }
  
  shutdown() {
    this.#shutdownController.abort('Client shutdown');
  }
}

// All pending requests cancelled on shutdown
const client = new ApiClient();
client.fetch('/api/data');
client.shutdown();  // Cancels all pending requests
```

## Summary

AbortController works with addEventListener (signal option removes listeners). Use for stream cancellation. throwIfAborted() for immediate check. Create shutdown controllers for batch operations. AbortSignal.any() combines user and system cancellation.

## Code Examples

**Event Listeners with Signal**

```javascript
const controller = new AbortController();

window.addEventListener('resize', handleResize, {
  signal: controller.signal
});

window.addEventListener('scroll', handleScroll, {
  signal: controller.signal  
});

// Remove ALL listeners with one call
controller.abort();
// No need for removeEventListener!
```

**Stream Cancellation**

```javascript
const controller = new AbortController();

async function processStream(url) {
  const response = await fetch(url, { signal: controller.signal });
  const reader = response.body.getReader();
  
  try {
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      processChunk(value);
    }
  } finally {
    reader.releaseLock();
  }
}

// Cancel mid-stream
controller.abort();
```


## Resources

- [MDN: EventTarget.addEventListener() signal](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#signal) — Using signal with event listeners

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*