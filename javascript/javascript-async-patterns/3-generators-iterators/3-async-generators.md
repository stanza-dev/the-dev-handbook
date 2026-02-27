---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-async-generators"
---

# Async Iterators & Generators

## Introduction

Async generators combine generators with promises, letting you yield async values. Combined with for await...of, they enable elegant streaming and pagination patterns that would be complex with promises alone.

## Key Concepts

**async function***: A generator that can use await and yields promises.

**for await...of**: Loop that awaits each iteration.

**Async Iterable**: Object with [Symbol.asyncIterator].

## Deep Dive

### Async Generator Syntax

```javascript
async function* fetchPages(url) {
  let page = 1;
  while (true) {
    const response = await fetch(`${url}?page=${page}`);
    const data = await response.json();
    
    if (data.length === 0) return;  // No more pages
    
    yield data;  // Yield the page data
    page++;
  }
}

// Consume with for await...of
async function processAllPages() {
  for await (const pageData of fetchPages('/api/items')) {
    console.log('Got page:', pageData.length, 'items');
  }
}
```

### for await...of Loop

```javascript
// Works with async iterables
const asyncIterable = {
  [Symbol.asyncIterator]() {
    let i = 0;
    return {
      async next() {
        if (i < 3) {
          await new Promise(r => setTimeout(r, 100));
          return { value: i++, done: false };
        }
        return { done: true };
      }
    };
  }
};

for await (const num of asyncIterable) {
  console.log(num);  // 0, 1, 2 (with 100ms delays)
}
```

### Practical Streaming Example

```javascript
async function* readChunks(stream) {
  const reader = stream.getReader();
  try {
    while (true) {
      const { done, value } = await reader.read();
      if (done) return;
      yield value;
    }
  } finally {
    reader.releaseLock();
  }
}

// Usage
const response = await fetch('/large-file');
for await (const chunk of readChunks(response.body)) {
  processChunk(chunk);
}
```

### Converting Callbacks to Async Iterator

```javascript
async function* socketMessages(socket) {
  const messages = [];
  let resolve;
  
  socket.on('message', (msg) => {
    messages.push(msg);
    resolve?.();
  });
  
  socket.on('close', () => resolve?.());
  
  while (socket.connected) {
    if (messages.length === 0) {
      await new Promise(r => resolve = r);
    }
    while (messages.length) {
      yield messages.shift();
    }
  }
}
```

## Common Pitfalls

1. **Using for...of instead of for await...of**: Won't await promises.
2. **Forgetting error handling**: Wrap in try/catch or handle in consumer.
3. **Memory buildup**: Don't buffer too much data.

## Best Practices

- **Use for await...of for async sequences**: Cleaner than manual iteration.
- **Handle errors in the loop**: Each iteration can fail.
- **Close resources in finally**: Readers, connections, etc.

## Summary

Async generators (async function*) combine await and yield. for await...of consumes async iterables. Great for pagination, streaming, and real-time data. Always handle cleanup in finally blocks.

## Resources

- [MDN: for await...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of) — for await...of reference
- [MDN: Async iteration](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/asyncIterator) — Async iterator protocol

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*