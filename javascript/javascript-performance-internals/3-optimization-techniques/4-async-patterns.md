---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-async-patterns"
---

## Introduction

Asynchronous JavaScript is the backbone of responsive web applications. However, poorly structured async code can block the main thread, cause memory leaks through unresolved promises, or serialize operations that could run in parallel. Understanding the nuances of the microtask queue, task scheduling, and modern async patterns is critical for building applications that remain responsive under heavy workloads.

This lesson covers advanced techniques for optimizing async operations, from parallel execution and task chunking to cancellation and streaming data with async generators.

## Key Concepts

- **Microtask Queue**: A high-priority queue where Promise callbacks (`.then`, `await` continuations) are processed. Microtasks are drained completely before the browser returns to the task queue or renders a frame, meaning a long chain of microtasks can block rendering.
- **Task Scheduling**: The process of breaking long-running work into smaller chunks and yielding control back to the browser between chunks, allowing the event loop to process user input and render frames.
- **Promise.allSettled**: Runs multiple promises in parallel and returns results for all of them, regardless of whether they fulfilled or rejected. Unlike `Promise.all`, it never short-circuits on a rejection.
- **AbortController**: A built-in API for signaling cancellation to async operations like `fetch`, event listeners, and custom async workflows. The controller's `signal` can be passed to any API that supports it.
- **Structured Concurrency**: A programming paradigm where async operations are grouped into scopes that ensure all child operations complete (or are cancelled) before the scope exits, preventing orphaned promises.

## Real World Context

Dashboards that fetch data from multiple microservices use `Promise.allSettled` to display partial results when some services are down. Search interfaces use `AbortController` to cancel in-flight requests when the user types a new query, preventing stale results from overwriting fresh ones. Long-running data processing (parsing large CSV files, image manipulation) uses task chunking to keep the UI responsive. Node.js servers use streaming and async generators to process large datasets without loading everything into memory.

## Deep Dive

When you need to run multiple independent async operations, `Promise.all` fails fast on the first rejection, which is often undesirable. `Promise.allSettled` gives you every result:

```javascript
const results = await Promise.allSettled([
  fetchUser(id),
  fetchOrders(id),
  fetchPreferences(id),
]);

const succeeded = results
  .filter(r => r.status === 'fulfilled')
  .map(r => r.value);

const failed = results
  .filter(r => r.status === 'rejected')
  .map(r => r.reason);
```

For long-running synchronous work, chunk the operation and yield to the main thread between chunks. The modern approach uses `scheduler.yield()` where available, falling back to `setTimeout`:

```javascript
async function processLargeArray(items) {
  const CHUNK = 100;
  for (let i = 0; i < items.length; i += CHUNK) {
    const chunk = items.slice(i, i + CHUNK);
    chunk.forEach(processItem);
    // Yield to the main thread
    if (globalThis.scheduler?.yield) {
      await scheduler.yield();
    } else {
      await new Promise(r => setTimeout(r, 0));
    }
  }
}
```

AbortController allows you to cancel fetch requests and custom async workflows:

```javascript
let controller;
async function search(query) {
  controller?.abort(); // Cancel previous request
  controller = new AbortController();
  try {
    const res = await fetch(`/api/search?q=${query}`, {
      signal: controller.signal,
    });
    return await res.json();
  } catch (e) {
    if (e.name === 'AbortError') return; // Expected cancellation
    throw e;
  }
}
```

Async generators allow you to produce values over time, processing data as a stream rather than waiting for the entire result:

```javascript
async function* fetchPages(url) {
  let nextUrl = url;
  while (nextUrl) {
    const res = await fetch(nextUrl);
    const data = await res.json();
    yield data.items;
    nextUrl = data.nextPage;
  }
}

for await (const page of fetchPages('/api/items')) {
  renderItems(page);
}
```

ES2025 Iterator Helpers bring lazy evaluation to iterators, enabling efficient data pipeline processing without creating intermediate arrays:

```javascript
// Instead of creating intermediate arrays:
const result = hugeArray
  .filter(x => x.active)    // Creates full intermediate array
  .map(x => x.value)        // Creates another intermediate array
  .slice(0, 10);            // Only needed 10 items

// With Iterator Helpers (lazy evaluation):
const result = hugeArray.values()
  .filter(x => x.active)    // Lazy — no intermediate array
  .map(x => x.value)        // Lazy — no intermediate array
  .take(10)                 // Stops after 10 items
  .toArray();               // Only materializes the final 10
```

You can also use `.drop()` to skip elements and chain these helpers with async iterators for streaming data pipelines.

## Common Pitfalls

- **Serializing independent operations**: Using `await` sequentially for operations that do not depend on each other (e.g., `await fetchA(); await fetchB();`) doubles the total wait time. Use `Promise.all` or `Promise.allSettled` for independent operations.
- **Microtask starvation**: A recursive chain of `Promise.resolve().then(...)` never yields to the task queue or renderer. If you need to yield to the browser, you must use `setTimeout(fn, 0)` or `scheduler.yield()`, not just `await Promise.resolve()`.
- **Forgetting to handle AbortError**: When using AbortController with fetch, the rejection is an `AbortError` that should be silently ignored, not treated as a real error. Failing to filter it out causes false error reports.

## Best Practices

- Use `Promise.allSettled` over `Promise.all` when partial success is acceptable and you need to handle each result independently.
- Always pass an `AbortSignal` to fetch requests that may become stale (search, autocomplete, navigation) to prevent wasted bandwidth and race conditions.
- For long-running computations in the browser, chunk work into batches of 5-10ms and yield to the main thread between batches to maintain 60fps responsiveness.

## Summary

Async performance optimization requires understanding the event loop's prioritization of microtasks over tasks. Use `Promise.allSettled` for parallel operations with error isolation, chunk long tasks to avoid blocking the main thread, and employ `AbortController` for cancellation. Async generators enable streaming data processing, and ES2025 Iterator Helpers provide lazy `.map()`, `.filter()`, `.take()`, and `.drop()` to build efficient data pipelines without intermediate array allocations.

## Code Examples

**Task chunking and parallel async patterns**

```javascript
// Chunk long tasks to avoid blocking
async function processLargeArray(items) {
  const CHUNK = 100;
  for (let i = 0; i < items.length; i += CHUNK) {
    const chunk = items.slice(i, i + CHUNK);
    chunk.forEach(processItem);
    // Yield to the main thread
    await new Promise(r => setTimeout(r, 0));
  }
}

// Parallel with error isolation
const results = await Promise.allSettled([
  fetchUser(id),
  fetchOrders(id),
  fetchPreferences(id),
]);
const succeeded = results
  .filter(r => r.status === 'fulfilled')
  .map(r => r.value);
```


## Resources

- [MDN: Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) — Promise patterns for async operations

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*