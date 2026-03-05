---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-blob-file"
---

## Introduction

While ArrayBuffer and TypedArrays give you direct access to bytes in memory, the **Blob** and **File** APIs operate at a higher level of abstraction. A Blob represents an immutable chunk of binary data that may not even reside in memory — it could be backed by a file on disk or data streamed from the network. The File API extends Blob for user-selected files. Together, they provide the tools for creating downloadable files, processing uploads, and streaming large binary data without loading everything into memory at once.

## Key Concepts

- **Blob**: An immutable, opaque object representing raw binary data. It has a `size` (in bytes) and a `type` (MIME type), but its contents are not directly accessible as a JavaScript array.
- **File**: A subclass of Blob that adds `name`, `lastModified`, and other metadata. Created by file input elements and drag-and-drop events.
- **FileReader**: An older, callback-based API for reading Blob/File contents as text, ArrayBuffer, or data URL. Largely superseded by `Blob.prototype.text()`, `arrayBuffer()`, and `stream()`.
- **URL.createObjectURL**: Creates a temporary URL pointing to a Blob's data, enabling it to be used as an `<img>` src, `<a>` href, or `<video>` source.
- **Blob Slicing**: `Blob.prototype.slice(start, end, type)` creates a new Blob referencing a byte range of the original without copying data.
- **Streams API**: `Blob.prototype.stream()` returns a `ReadableStream` of `Uint8Array` chunks, enabling incremental processing of large files.

## Real World Context

Blob and File are everywhere in web applications. File upload features use the File API to validate and preview user-selected files before sending them to a server. Client-side file generation — exporting CSV reports, saving canvas artwork as PNG, or generating PDF documents — relies on Blob creation and download. Large file processors like video editors and ZIP utilities use `Blob.stream()` to handle multi-gigabyte files without exhausting memory. Understanding these APIs is essential for any application that creates, consumes, or transforms files in the browser.

## Deep Dive

### Creating Blobs from Data

The `Blob` constructor accepts an array of parts (strings, ArrayBuffers, TypedArrays, or other Blobs) and an optional options object with a `type` property:

```javascript
// From a string
const textBlob = new Blob(['Hello, world!'], { type: 'text/plain' });

// From JSON
const jsonBlob = new Blob(
  [JSON.stringify({ key: 'value' })],
  { type: 'application/json' }
);

// From binary data
const bytes = new Uint8Array([0x89, 0x50, 0x4E, 0x47]);
const binaryBlob = new Blob([bytes], { type: 'image/png' });

// From multiple parts
const multiBlob = new Blob(['Header\n', bytes, '\nFooter']);
```

### Reading Blob Contents

Modern APIs provide promise-based methods directly on Blob:

```javascript
const text = await blob.text();           // String
const buffer = await blob.arrayBuffer();  // ArrayBuffer
const stream = blob.stream();             // ReadableStream
```

The older `FileReader` API uses events and is only needed for legacy browser support or for reading as a data URL (`readAsDataURL`).

### File API for User Uploads

The `File` object extends `Blob` with file metadata. You get `File` objects from `<input type="file">` elements and drag-and-drop events:

```javascript
input.addEventListener('change', (e) => {
  const file = e.target.files[0]; // File object
  console.log(file.name);         // 'photo.jpg'
  console.log(file.size);         // bytes
  console.log(file.type);         // 'image/jpeg'
  console.log(file.lastModified); // timestamp
});
```

### Streaming Large Files

For large files, loading the entire contents into memory is impractical. `Blob.stream()` returns a `ReadableStream` that yields `Uint8Array` chunks, enabling incremental processing:

```javascript
async function hashFile(file) {
  const stream = file.stream();
  const reader = stream.getReader();
  let totalBytes = 0;
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    totalBytes += value.byteLength;
    // Process each chunk incrementally
  }
  return totalBytes;
}
```

### Blob URLs and Memory Management

`URL.createObjectURL(blob)` creates a `blob:` URL that can be used anywhere a regular URL is expected. This is how you display uploaded images, trigger file downloads, or set video sources without a server round-trip:

```javascript
const url = URL.createObjectURL(blob);
img.src = url; // Display the blob as an image
```

Critically, blob URLs **hold a strong reference** to the Blob, preventing garbage collection. You must call `URL.revokeObjectURL(url)` when the URL is no longer needed to release the memory.

### Blob Slicing

`blob.slice(start, end, contentType)` creates a new Blob referencing a byte range of the original. This is a zero-copy operation — no data is duplicated:

```javascript
const header = largeBlob.slice(0, 1024, 'application/octet-stream');
const chunk = largeBlob.slice(1024, 2048);
```

This is useful for chunked uploads, where you send a large file in pieces, and for parsing file headers without reading the entire file.

## Common Pitfalls

- **Leaking Blob URLs**: Every `createObjectURL` call allocates a URL that persists until revoked or the document unloads. In single-page applications with frequent file previews, this causes steady memory growth. Always pair `createObjectURL` with `revokeObjectURL`.
- **Blocking the main thread with large reads**: Calling `blob.arrayBuffer()` on a multi-gigabyte file will attempt to allocate that much memory at once, potentially crashing the tab. Use streaming for large files.
- **Assuming Blob is synchronous**: Blob contents are not immediately accessible. All read operations are asynchronous. Attempting to use a Blob like an ArrayBuffer will fail.

## Best Practices

- **Use `Blob.stream()` for files over a few megabytes.** Streaming avoids memory spikes and enables progress reporting.
- **Revoke Blob URLs immediately after use.** In a download scenario, revoke the URL right after triggering the click. For image previews, revoke in the `onload` handler.
- **Prefer the modern promise-based Blob methods** (`text()`, `arrayBuffer()`, `stream()`) over `FileReader`. They produce cleaner code and integrate naturally with `async/await`.

## Summary

The Blob API provides an immutable, high-level interface for binary data that abstracts away memory management details. The File API extends it for user-selected files. Together with `URL.createObjectURL` for display, `Blob.stream()` for incremental processing, and `slice()` for zero-copy subsetting, they form a complete toolkit for client-side file creation, upload handling, and large-data processing. Always revoke Blob URLs to prevent memory leaks, and prefer streaming over bulk reads for large files.

## Code Examples

**Blob creation, download, and streaming large files**

```javascript
// Create and download a file
const data = JSON.stringify({ name: 'test' }, null, 2);
const blob = new Blob([data], { type: 'application/json' });
const url = URL.createObjectURL(blob);

const a = document.createElement('a');
a.href = url;
a.download = 'data.json';
a.click();
URL.revokeObjectURL(url); // Free memory

// Stream a large file
async function processLargeFile(file) {
  const stream = file.stream();
  const reader = stream.getReader();
  let totalBytes = 0;
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    totalBytes += value.byteLength;
    processChunk(value); // Uint8Array chunk
  }
  console.log('Processed', totalBytes, 'bytes');
}
```


## Resources

- [MDN: Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob) — Blob API for immutable binary data
- [MDN: File API](https://developer.mozilla.org/en-US/docs/Web/API/File_API) — File API for accessing user-selected files

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*