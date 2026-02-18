---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-binary-use-cases"
---

# Binary Data Use Cases

## Introduction

TypedArrays are essential for graphics, audio, and file processing.

## Deep Dive

### Image Processing

```javascript
const ctx = canvas.getContext('2d');
const imageData = ctx.getImageData(0, 0, w, h);

// Uint8ClampedArray: [R,G,B,A, R,G,B,A, ...]
for (let i = 0; i < imageData.data.length; i += 4) {
  const avg = (imageData.data[i] + imageData.data[i+1] + imageData.data[i+2]) / 3;
  imageData.data[i] = avg;     // R
  imageData.data[i+1] = avg;   // G
  imageData.data[i+2] = avg;   // B
}
ctx.putImageData(imageData, 0, 0);
```

### WebGL Buffers

```javascript
const positions = new Float32Array([
  -0.5, -0.5, 0.0,
   0.5, -0.5, 0.0,
   0.0,  0.5, 0.0
]);

gl.bufferData(gl.ARRAY_BUFFER, positions, gl.STATIC_DRAW);
```

### Audio Processing

```javascript
const samples = audioBuffer.getChannelData(0); // Float32Array
for (let i = 0; i < samples.length; i++) {
  samples[i] *= 0.5;  // Reduce volume
}
```

## Summary

TypedArrays for Canvas pixels, WebGL, Web Audio. Much faster than regular arrays for numeric work.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*