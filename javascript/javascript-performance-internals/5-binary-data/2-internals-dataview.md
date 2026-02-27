---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-dataview"
---

# DataView for Mixed Data

## Introduction

DataView provides flexible access with explicit endianness control and mixed data types.

## Deep Dive

### Basic Usage

```javascript
const buffer = new ArrayBuffer(12);
const view = new DataView(buffer);

view.setInt32(0, 12345);       // Bytes 0-3
view.setFloat32(4, 3.14159);   // Bytes 4-7
view.setUint8(8, 255);         // Byte 8

view.getInt32(0);   // 12345
view.getFloat32(4); // 3.14159
view.getUint8(8);   // 255
```

### Endianness

```javascript
view.setInt32(0, 0x12345678, true);  // Little-endian
view.setInt32(0, 0x12345678, false); // Big-endian

// Network protocols often use big-endian
// x86 CPUs use little-endian
```

### Parsing Binary Protocols

```javascript
function parsePacket(buffer) {
  const view = new DataView(buffer);
  return {
    version: view.getUint8(0),
    type: view.getUint8(1),
    length: view.getUint16(2),
    timestamp: view.getFloat64(4)
  };
}
```

## Summary

DataView for mixed-type binary data. Explicit endianness control. Better for parsing protocols.

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*