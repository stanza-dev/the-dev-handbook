---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-transform-streams"
---

# Transform Streams & Piping

## Introduction

TransformStreams process data as it flows through—decompressing, parsing, filtering. Combined with piping, they create powerful data processing pipelines without loading everything into memory.

## Key Concepts

**TransformStream**: Has both a writable side (input) and readable side (output).

**Piping**: Connecting streams together.

**Backpressure**: Automatic flow control when destination is slow.

## Deep Dive

### Basic Transform

```javascript
const uppercaseTransform = new TransformStream({
  transform(chunk, controller) {
    controller.enqueue(chunk.toUpperCase());
  }
});

// Using with text
const response = await fetch('/text-data');
const uppercased = response.body
  .pipeThrough(new TextDecoderStream())
  .pipeThrough(uppercaseTransform);

for await (const chunk of uppercased) {
  console.log(chunk);
}
```

### Built-in Transform Streams

```javascript
// Text encoding/decoding
const textDecoder = new TextDecoderStream('utf-8');
const textEncoder = new TextEncoderStream();

// Compression (Chrome)
const compressor = new CompressionStream('gzip');
const decompressor = new DecompressionStream('gzip');

// Example: compress and download
const response = await fetch('/large-data');
const compressed = response.body.pipeThrough(new CompressionStream('gzip'));
// Save compressed stream...
```

### Piping Streams Together

```javascript
// pipeTo - consumes readable into writable
readable.pipeTo(writable);

// pipeThrough - returns new readable
const transformed = readable.pipeThrough(transform);

// Chain multiple transforms
const result = inputStream
  .pipeThrough(new TextDecoderStream())
  .pipeThrough(jsonLineParser)   // Custom transform
  .pipeThrough(filter(x => x.active))
  .pipeTo(destination);
```

### Custom JSON Lines Parser

```javascript
function createJsonLinesParser() {
  let buffer = '';
  
  return new TransformStream({
    transform(chunk, controller) {
      buffer += chunk;
      const lines = buffer.split('\n');
      buffer = lines.pop();
      
      for (const line of lines) {
        if (line.trim()) {
          controller.enqueue(JSON.parse(line));
        }
      }
    },
    flush(controller) {
      if (buffer.trim()) {
        controller.enqueue(JSON.parse(buffer));
      }
    }
  });
}

// Stream JSON lines file
const response = await fetch('/events.ndjson');
const events = response.body
  .pipeThrough(new TextDecoderStream())
  .pipeThrough(createJsonLinesParser());

for await (const event of events) {
  processEvent(event);
}
```

## Summary

TransformStream processes data as it passes through. Use pipeThrough for transforms, pipeTo for destinations. Built-in: TextDecoderStream, CompressionStream. Create custom transforms for parsing, filtering, mapping. Backpressure is handled automatically.

## Code Examples

**Basic Transform**

```javascript
const uppercaseTransform = new TransformStream({
  transform(chunk, controller) {
    controller.enqueue(chunk.toUpperCase());
  }
});

// Using with text
const response = await fetch('/text-data');
const uppercased = response.body
  .pipeThrough(new TextDecoderStream())
  .pipeThrough(uppercaseTransform);

for await (const chunk of uppercased) {
  console.log(chunk);
}
```

**Built-in Transform Streams**

```javascript
// Text encoding/decoding
const textDecoder = new TextDecoderStream('utf-8');
const textEncoder = new TextEncoderStream();

// Compression (Chrome)
const compressor = new CompressionStream('gzip');
const decompressor = new DecompressionStream('gzip');

// Example: compress and download
const response = await fetch('/large-data');
const compressed = response.body.pipeThrough(new CompressionStream('gzip'));
// Save compressed stream...
```


## Resources

- [MDN: TransformStream](https://developer.mozilla.org/en-US/docs/Web/API/TransformStream) — TransformStream reference
- [MDN: Streams API concepts](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API/Concepts) — Streams concepts and usage

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*