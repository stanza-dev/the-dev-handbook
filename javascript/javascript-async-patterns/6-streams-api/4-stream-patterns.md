---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-stream-patterns"
---

# Practical Stream Patterns

## Introduction

Streams shine in specific scenarios. This lesson covers practical patterns you'll use in real applications.

## Deep Dive

### Server-Sent Events with Streams

```javascript
async function* readSSE(url) {
  const response = await fetch(url);
  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = '';
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    buffer += decoder.decode(value, { stream: true });
    
    // SSE format: "data: ...\n\n"
    const messages = buffer.split('\n\n');
    buffer = messages.pop();
    
    for (const msg of messages) {
      if (msg.startsWith('data: ')) {
        yield JSON.parse(msg.slice(6));
      }
    }
  }
}

for await (const event of readSSE('/api/stream')) {
  console.log('Event:', event);
}
```

### Streaming Upload Progress

```javascript
async function uploadWithProgress(url, file, onProgress) {
  const totalSize = file.size;
  let uploadedSize = 0;
  
  const trackingStream = new TransformStream({
    transform(chunk, controller) {
      uploadedSize += chunk.byteLength;
      onProgress(uploadedSize / totalSize);
      controller.enqueue(chunk);
    }
  });
  
  await fetch(url, {
    method: 'POST',
    body: file.stream().pipeThrough(trackingStream),
    duplex: 'half'  // Required for streaming body
  });
}
```

### Tee for Multiple Consumers

```javascript
const response = await fetch('/data');

// tee() creates two independent streams from one
const [stream1, stream2] = response.body.tee();

// One stream for display
showInUI(stream1);

// One stream for storage
saveToCache(stream2);
```

### Stream from Async Generator

```javascript
function streamFromGenerator(asyncGen) {
  return new ReadableStream({
    async pull(controller) {
      const { value, done } = await asyncGen.next();
      if (done) {
        controller.close();
      } else {
        controller.enqueue(value);
      }
    }
  });
}

async function* generateData() {
  for (let i = 0; i < 100; i++) {
    await delay(10);
    yield `Item ${i}\n`;
  }
}

const stream = streamFromGenerator(generateData());
```

### Memory-Efficient Large File Processing

```javascript
async function processLargeCSV(file) {
  const results = [];
  
  const parser = new TransformStream({
    buffer: '',
    transform(chunk, controller) {
      this.buffer += chunk;
      const lines = this.buffer.split('\n');
      this.buffer = lines.pop();
      for (const line of lines) {
        controller.enqueue(parseCSVLine(line));
      }
    },
    flush(controller) {
      if (this.buffer) {
        controller.enqueue(parseCSVLine(this.buffer));
      }
    }
  });
  
  const readable = file.stream()
    .pipeThrough(new TextDecoderStream())
    .pipeThrough(parser);
  
  for await (const row of readable) {
    processRow(row);  // Process row-by-row
  }
}
```

## Summary

Use streams for SSE parsing, upload progress, large file processing. tee() duplicates streams for multiple consumers. Convert async generators to streams. Process large files row-by-row to avoid memory issues.

## Code Examples

**Server-Sent Events with Streams**

```javascript
async function* readSSE(url) {
  const response = await fetch(url);
  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = '';
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    buffer += decoder.decode(value, { stream: true });
    
    // SSE format: "data: ...\n\n"
    const messages = buffer.split('\n\n');
    buffer = messages.pop();
    
    for (const msg of messages) {
      if (msg.startsWith('data: ')) {
```

**Streaming Upload Progress**

```javascript
async function uploadWithProgress(url, file, onProgress) {
  const totalSize = file.size;
  let uploadedSize = 0;
  
  const trackingStream = new TransformStream({
    transform(chunk, controller) {
      uploadedSize += chunk.byteLength;
      onProgress(uploadedSize / totalSize);
      controller.enqueue(chunk);
    }
  });
  
  await fetch(url, {
    method: 'POST',
    body: file.stream().pipeThrough(trackingStream),
    duplex: 'half'  // Required for streaming body
  });
}
```


## Resources

- [MDN: Using readable streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API/Using_readable_streams) — Guide to using readable streams

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*