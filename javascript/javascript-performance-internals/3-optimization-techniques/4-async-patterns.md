---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-async-patterns"
---

# Async Performance Patterns

## Introduction

Efficient async code parallelizes work, chunks long tasks, and keeps the UI responsive.

## Deep Dive

### Parallelize Independent Operations

```javascript
// Bad: Sequential (slow)
const user = await fetchUser();
const posts = await fetchPosts();
const comments = await fetchComments();

// Good: Parallel (fast)
const [user, posts, comments] = await Promise.all([
  fetchUser(), fetchPosts(), fetchComments()
]);
```

### Chunking Long Tasks

```javascript
async function processChunked(items, chunkSize = 100) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    chunk.forEach(process);
    await new Promise(r => setTimeout(r, 0));  // Yield
  }
}
```

### requestIdleCallback

```javascript
function doBackgroundWork(deadline) {
  while (deadline.timeRemaining() > 0 && queue.length) {
    processItem(queue.shift());
  }
  if (queue.length) requestIdleCallback(doBackgroundWork);
}

requestIdleCallback(doBackgroundWork);
```

## Summary

Parallelize independent async ops. Chunk long tasks. Use requestIdleCallback for background work.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*