---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-blob-file"
---

# Blob and File APIs

## Introduction

Blob represents immutable raw data. File extends Blob with metadata.

## Deep Dive

### Creating Blobs

```javascript
const textBlob = new Blob(['Hello'], { type: 'text/plain' });
const binaryBlob = new Blob([buffer], { type: 'application/octet-stream' });
```

### Reading Blobs

```javascript
const buffer = await blob.arrayBuffer();
const text = await blob.text();
const stream = blob.stream();
```

### File Download

```javascript
function download(blob, filename) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}
```

### File Upload Processing

```javascript
input.addEventListener('change', async (e) => {
  const file = e.target.files[0];
  const buffer = await file.arrayBuffer();
  const bytes = new Uint8Array(buffer);
});
```

## Summary

Blob for immutable binary. File adds metadata. arrayBuffer() for raw bytes. URL.createObjectURL for downloads.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*