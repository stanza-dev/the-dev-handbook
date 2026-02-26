---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-readable-streams"
---

# Readable Streams

## Introduction

Streams let you process data as it arrives rather than waiting for everything. For large files, real-time data, or memory-constrained environments, streams are essential. The Streams API provides a standard way to work with streaming data.

## Key Concepts

**ReadableStream**: A source of data you can read from.

**Chunk**: A piece of data from the stream (usually Uint8Array).

**Reader**: Object that reads chunks from a stream.

## Real World Context

Streaming video, processing large CSV files, real-time log viewing, upload progress—anywhere data is too large or too slow to load all at once.

## Deep Dive

### Reading a Fetch Response Stream

```javascript
const response = await fetch('/large-file');
const reader = response.body.getReader();

let totalBytes = 0;

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  
  totalBytes += value.byteLength;  // value is Uint8Array
  console.log(`Received ${value.byteLength} bytes`);
}

console.log(`Total: ${totalBytes} bytes`);
```

### Reading as Text

```javascript
async function* streamLines(response) {
  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = '';
  
  while (true) {
    const { done, value } = await reader.read();
    
    if (done) {
      if (buffer) yield buffer;
      return;
    }
    
    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split('\n');
    buffer = lines.pop();  // Keep incomplete line
    
    for (const line of lines) {
      yield line;
    }
  }
}

const response = await fetch('/logs.txt');
for await (const line of streamLines(response)) {
  console.log(line);
}
```

### Stream with Progress

```javascript
async function fetchWithProgress(url, onProgress) {
  const response = await fetch(url);
  const contentLength = +response.headers.get('Content-Length');
  const reader = response.body.getReader();
  
  const chunks = [];
  let receivedLength = 0;
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    chunks.push(value);
    receivedLength += value.length;
    onProgress(receivedLength / contentLength);
  }
  
  // Combine chunks
  const all = new Uint8Array(receivedLength);
  let position = 0;
  for (const chunk of chunks) {
    all.set(chunk, position);
    position += chunk.length;
  }
  
  return new TextDecoder().decode(all);
}
```

### Using Response Methods (Auto-Streams)

```javascript
// These consume the stream for you
const response = await fetch('/api/data');

await response.json();     // Reads stream, parses JSON
await response.text();     // Reads stream as text
await response.blob();     // Reads stream as Blob
await response.arrayBuffer(); // Reads stream as ArrayBuffer

// Note: Can only consume once!
// response.text() after response.json() throws
```

## Common Pitfalls

1. **Consuming stream twice**: Can only read once.
2. **Forgetting to release reader**: Use try/finally or releaseLock().
3. **TextDecoder without `stream: true`**: Corrupts multi-byte characters.

## Best Practices

- **Use for large data**: When loading all at once would be problematic.
- **Release locks**: Call reader.releaseLock() when done.
- **Use TextDecoder with stream option**: For proper multi-byte handling.
- **Consider libraries**: For complex streaming like ndjson.

## Summary

ReadableStream lets you process data incrementally. Get a reader with getReader(), read chunks in a loop, check done to know when finished. Chunks are Uint8Array by default. Use TextDecoder for text. Can only consume once.

## Code Examples

**Reading a Fetch Response Stream**

```javascript
const response = await fetch('/large-file');
const reader = response.body.getReader();

let totalBytes = 0;

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  
  totalBytes += value.byteLength;  // value is Uint8Array
  console.log(`Received ${value.byteLength} bytes`);
}

console.log(`Total: ${totalBytes} bytes`);
```

**Reading as Text**

```javascript
async function* streamLines(response) {
  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = '';
  
  while (true) {
    const { done, value } = await reader.read();
    
    if (done) {
      if (buffer) yield buffer;
      return;
    }
    
    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split('\n');
    buffer = lines.pop();  // Keep incomplete line
    
    for (const line of lines) {
```


## Resources

- [MDN: Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) — Streams API overview
- [MDN: ReadableStream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) — ReadableStream reference

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*