---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-writable-streams"
---

# Writable Streams

## Introduction

Writable streams are destinations for data. While less common in browser JavaScript, they're essential for Node.js file operations and advanced browser APIs like File System Access.

## Key Concepts

**WritableStream**: A destination you can write data to.

**Writer**: Object used to write chunks to the stream.

**Backpressure**: When the destination can't keep up with incoming data.

## Deep Dive

### Basic Writing

```javascript
const writableStream = new WritableStream({
  write(chunk) {
    console.log('Writing:', chunk);
  },
  close() {
    console.log('Stream closed');
  },
  abort(err) {
    console.error('Stream aborted:', err);
  }
});

const writer = writableStream.getWriter();
await writer.write('Hello');
await writer.write('World');
await writer.close();
```

### File System Access API

```javascript
// Save streamed data to file (Chrome)
const fileHandle = await window.showSaveFilePicker();
const writable = await fileHandle.createWritable();

// Stream response to file
const response = await fetch('/large-file');
await response.body.pipeTo(writable);
// File is automatically closed
```

### Buffered Writing

```javascript
const { writable, readable } = new TransformStream();

// Writer with backpressure handling
const writer = writable.getWriter();

for (const chunk of largeData) {
  // desiredSize tells you how much buffer room is left
  if (writer.desiredSize <= 0) {
    await writer.ready;  // Wait for drain
  }
  await writer.write(chunk);
}
await writer.close();
```

## Summary

WritableStream is a data destination. Use getWriter() to write. await writer.ready for backpressure. pipeTo() connects readable to writable. Important for file operations and custom data processing.

## Resources

- [MDN: WritableStream](https://developer.mozilla.org/en-US/docs/Web/API/WritableStream) — WritableStream reference

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*