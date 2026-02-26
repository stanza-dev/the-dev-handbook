---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-binary-use-cases"
---

## Introduction

Understanding ArrayBuffer, TypedArrays, and DataView is only half the story. The real power of binary data in JavaScript comes from applying these primitives to solve concrete problems across the web platform. From manipulating individual pixels on a canvas to feeding vertex data into a GPU, typed arrays are the connective tissue between JavaScript and the high-performance APIs that make modern web experiences possible.

## Key Concepts

- **WebGL Buffers**: WebGL uses `Float32Array` to pass vertex positions, normals, and texture coordinates to the GPU for 3D rendering.
- **Web Audio**: The Web Audio API processes audio samples as `Float32Array` buffers, enabling real-time audio synthesis, filtering, and analysis.
- **Canvas Pixel Manipulation**: The Canvas 2D API exposes pixel data as a `Uint8ClampedArray`, allowing direct per-pixel read and write operations on images.
- **WebSocket Binary**: WebSockets can transmit `ArrayBuffer` data for compact, efficient binary protocols instead of JSON strings.
- **Encoding API**: `TextEncoder` and `TextDecoder` convert between JavaScript strings and UTF-8 byte sequences stored in `Uint8Array`.

## Real World Context

Every image filter app, 3D game, audio visualizer, and real-time collaboration tool on the web relies on binary data APIs. Instagram-style photo filters manipulate canvas pixels through `Uint8ClampedArray`. Three.js and Babylon.js pack geometry into `Float32Array` for WebGL rendering. Music production apps like Soundtrap process audio through `Float32Array` buffers in AudioWorklets. Chat applications use binary WebSocket frames to transmit protocol buffers. Understanding these use cases transforms typed arrays from abstract concepts into practical tools.

## Deep Dive

### Canvas Pixel Manipulation with Uint8ClampedArray

The Canvas 2D API's `getImageData()` returns an `ImageData` object whose `data` property is a `Uint8ClampedArray`. Each pixel occupies 4 consecutive bytes: Red, Green, Blue, Alpha (RGBA). The "clamped" behavior means values are automatically constrained to 0–255 without wrapping — adding 10 to 250 yields 255, not 4:

```javascript
const imageData = ctx.getImageData(0, 0, width, height);
const pixels = imageData.data; // Uint8ClampedArray
// Pixel at (x, y) starts at index: (y * width + x) * 4
```

Common operations include grayscale conversion (averaging RGB channels), brightness adjustment (adding/subtracting from each channel), and threshold filters (setting pixels to black or white based on luminance).

### WebGL Vertex Buffers with Float32Array

WebGL requires geometry data packed into typed arrays before uploading to the GPU:

```javascript
const vertices = new Float32Array([
  -1.0, -1.0, 0.0,  // vertex 1 (x, y, z)
   1.0, -1.0, 0.0,  // vertex 2
   0.0,  1.0, 0.0   // vertex 3
]);
gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
```

`Float32Array` is the standard because GPUs natively operate on 32-bit floats. Using `Float64Array` would waste bandwidth — the GPU would need to downcast every value.

### Web Audio Processing with Float32Array

The Web Audio API represents audio samples as 32-bit floats in the range [-1.0, 1.0]. An `AudioWorkletProcessor` receives `Float32Array` input buffers and writes to `Float32Array` output buffers on every audio processing quantum (typically 128 samples):

```javascript
class GainProcessor extends AudioWorkletProcessor {
  process(inputs, outputs) {
    const input = inputs[0][0];   // Float32Array
    const output = outputs[0][0]; // Float32Array
    for (let i = 0; i < input.length; i++) {
      output[i] = input[i] * 0.5; // Reduce volume by half
    }
    return true;
  }
}
```

### Text Encoding and Decoding

`TextEncoder` converts a JavaScript string to UTF-8 bytes in a `Uint8Array`. `TextDecoder` performs the reverse. This is essential for binary protocols that embed text fields, for hashing strings with the SubtleCrypto API, and for working with binary file formats that contain text headers:

```javascript
const encoder = new TextEncoder();
const bytes = encoder.encode('Hello, world!'); // Uint8Array
const decoder = new TextDecoder('utf-8');
const text = decoder.decode(bytes); // 'Hello, world!'
```

## Common Pitfalls

- **Forgetting to call `putImageData()` after modifying pixels.** Modifying the `Uint8ClampedArray` does not automatically update the canvas — you must write the data back explicitly.
- **Using the wrong typed array type for an API.** WebGL expects `Float32Array` for most buffer data. Passing a `Float64Array` will either fail silently or produce garbage rendering.
- **Assuming TextDecoder defaults to UTF-8 everywhere.** While UTF-8 is the default, some binary formats use other encodings (e.g., UTF-16LE for Windows file metadata). Always specify the encoding explicitly when parsing unknown data.

## Best Practices

- **Process pixels in bulk using typed array methods** like `set()` and `subarray()` instead of element-by-element loops when possible. The engine can optimize bulk operations more aggressively.
- **Reuse typed array buffers across frames** in animation and audio processing loops. Allocating new arrays every frame triggers garbage collection pauses that cause visible jank or audio glitches.
- **Use `TextEncoder.encodeInto()` for zero-copy encoding** into a pre-allocated buffer when performance matters, avoiding the allocation of a new `Uint8Array` on each call.

## Summary

Typed arrays are the universal interface between JavaScript and the web platform's high-performance APIs. `Uint8ClampedArray` powers canvas pixel manipulation, `Float32Array` feeds data to WebGL and Web Audio, and `TextEncoder`/`TextDecoder` bridge the gap between strings and bytes. Mastering these use cases turns binary data primitives into practical tools for building image editors, 3D renderers, audio processors, and efficient network protocols.

## Code Examples

**Canvas pixel manipulation and text encoding with typed arrays**

```javascript
// Canvas pixel manipulation
const ctx = canvas.getContext('2d');
const imageData = ctx.getImageData(0, 0, width, height);
const pixels = imageData.data; // Uint8ClampedArray [R,G,B,A,...]

// Invert colors
for (let i = 0; i < pixels.length; i += 4) {
  pixels[i]     = 255 - pixels[i];     // R
  pixels[i + 1] = 255 - pixels[i + 1]; // G
  pixels[i + 2] = 255 - pixels[i + 2]; // B
  // pixels[i + 3] is Alpha — leave unchanged
}
ctx.putImageData(imageData, 0, 0);

// Text encoding
const encoder = new TextEncoder();
const bytes = encoder.encode('Hello'); // Uint8Array
const decoder = new TextDecoder();
console.log(decoder.decode(bytes)); // 'Hello'
```


## Resources

- [MDN: Canvas Pixel Manipulation](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas) — Manipulating canvas pixels with typed arrays

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*