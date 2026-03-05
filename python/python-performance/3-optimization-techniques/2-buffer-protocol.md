---
source_course: "python-performance"
source_lesson: "python-performance-buffer-protocol"
---

# The Buffer Protocol

## Introduction

Copying data between objects is one of the most common hidden performance costs in Python. The buffer protocol allows objects to expose their underlying memory directly, enabling zero-copy operations through `memoryview` and efficient binary manipulation with `struct` and `array`.

## Key Concepts

- **Buffer protocol**: A C-level API that lets objects expose raw memory regions for direct access without copying.
- **`memoryview`**: A Python wrapper that provides a zero-copy window into buffer-compatible objects like `bytes`, `bytearray`, and `array.array`.
- **`struct` module**: Converts between Python values and C-style packed binary data, essential for binary file formats and network protocols.
- **`array` module**: Provides typed arrays that store elements as contiguous C-type values, using far less memory than lists for numeric data.

## Real World Context

Network servers processing binary protocols (DNS, HTTP/2 frames, WebSocket), image processing pipelines manipulating pixel buffers, and scientific computing working with large numeric arrays all rely on the buffer protocol to avoid copying megabytes of data on every operation.

## Deep Dive

### Zero-Copy with memoryview

When you slice a `bytes` or `bytearray`, Python creates a new copy. A `memoryview` slice points to the same memory:

```python
data = bytearray(b"Hello, World!")

# Regular slice creates a copy
copy_slice = data[7:]       # New bytearray allocated

# memoryview slice shares memory
view = memoryview(data)
zero_copy_slice = view[7:]  # No copy, same underlying buffer
```

You can verify they share memory by mutating through the view:

```python
data = bytearray(b"Hello, World!")
view = memoryview(data)

view[0] = ord('h')          # Modify through the view
print(data)                  # bytearray(b'hello, World!') — original changed

sub = view[7:12]
sub[0] = ord('w')
print(data)                  # bytearray(b'hello, world!') — still the same buffer
```

This is critical when processing large buffers. Parsing a 10 MB binary file by slicing views instead of copying bytes saves both time and memory.

### The struct Module for Binary Data

The `struct` module packs and unpacks data into fixed-size binary formats:

```python
import struct

# Pack an unsigned byte, unsigned short, and unsigned int
header = struct.pack('>BHI', 1, 512, 1048576)  # Big-endian
print(len(header))  # 7 bytes total

# Unpack back to Python values
version, flags, size = struct.unpack('>BHI', header)
print(f"version={version}, flags={flags}, size={size}")
```

Use `struct.unpack_from` with a `memoryview` for zero-copy parsing of large buffers:

```python
import struct

buffer = bytearray(1024)  # Imagine this is a file or network packet
view = memoryview(buffer)

# Parse header at offset 0 without copying
magic, version, payload_size = struct.unpack_from('<4sHI', view, offset=0)
```

### The array Module

The `array` module stores elements as contiguous typed values, like a C array:

```python
import array
import sys

# A list of 10,000 integers
int_list = list(range(10000))
print(sys.getsizeof(int_list))  # ~85,000 bytes (pointers + int objects)

# An array of 10,000 signed integers
int_array = array.array('i', range(10000))
print(sys.getsizeof(int_array))  # ~40,000 bytes (raw 4-byte ints)
```

Arrays support the buffer protocol, so you can wrap them with `memoryview` for zero-copy slicing.

### NumPy Integration

NumPy arrays also implement the buffer protocol, allowing seamless zero-copy interop:

```python
import numpy as np

raw = bytearray(32)
np_view = np.frombuffer(raw, dtype=np.float64)  # 4 doubles, shared memory

np_view[0] = 3.14
print(raw[:8])  # The underlying bytes changed
```

## Common Pitfalls

- **Forgetting that memoryview holds a reference**: The underlying buffer cannot be resized while a memoryview exists. Calling `bytearray.extend()` while a view is active raises `BufferError`.
- **Using the wrong byte order in struct**: Network protocols typically use big-endian (`>`), while x86 systems are little-endian (`<`). Mismatching causes corrupt values without any error.
- **Assuming array.array is always faster than list**: For small collections or mixed-type data, the overhead of type conversion can make `array` slower. It shines for large homogeneous numeric data.

## Best Practices

- Use `memoryview` when parsing or processing large binary buffers to avoid unnecessary copies.
- Always specify explicit byte order in `struct` format strings (`<` for little-endian, `>` for big-endian, `=` for native) rather than relying on defaults.
- Prefer `struct.unpack_from` with offset over slicing the buffer first — it avoids an intermediate copy.

## Summary

- The buffer protocol lets objects expose raw memory, enabling zero-copy access through `memoryview`.
- `memoryview` slices share the same underlying buffer, unlike regular byte slices which always copy.
- The `struct` module converts between Python values and packed binary data for file/network protocols.
- The `array` module provides typed, contiguous storage that uses 2-4x less memory than lists for numeric data.
- NumPy integrates with the buffer protocol for seamless zero-copy interop with native Python buffers.

## Code Examples

**Parsing a binary network packet header with struct.unpack_from and memoryview for zero-copy payload access**

```python
import struct

def parse_packet(data: bytes) -> dict:
    """Parse a binary network packet using memoryview for zero-copy access."""
    view = memoryview(data)

    # Header: 1-byte version, 2-byte type, 4-byte payload length (big-endian)
    version, pkt_type, payload_len = struct.unpack_from('>BHI', view, 0)

    # Payload: zero-copy slice
    header_size = struct.calcsize('>BHI')
    payload = view[header_size:header_size + payload_len]

    return {
        'version': version,
        'type': pkt_type,
        'payload_length': payload_len,
        'payload': bytes(payload)
    }

# Build a test packet
test_payload = b"Hello, buffer protocol!"
packet = struct.pack('>BHI', 1, 42, len(test_payload)) + test_payload
result = parse_packet(packet)
print(result)
```

**Comparing memory usage between a Python list and array.array for homogeneous float data**

```python
import array
import sys

# Compare memory: list vs array for 100,000 floats
float_list = [float(i) for i in range(100_000)]
float_array = array.array('d', (float(i) for i in range(100_000)))

list_size = sys.getsizeof(float_list) + sum(sys.getsizeof(x) for x in float_list[:10]) * 10_000
array_size = sys.getsizeof(float_array)

print(f"list approx: {list_size / 1024 / 1024:.1f} MB")
print(f"array.array: {array_size / 1024 / 1024:.1f} MB")
print(f"array supports buffer: {hasattr(float_array, '__buffer__')}")

# Zero-copy slice via memoryview
view = memoryview(float_array)
sub = view[:10]  # No copy
print(f"First 10 via view: {sub.tolist()}")
```


## Resources

- [memoryview — Built-in Types](https://docs.python.org/3.14/library/stdtypes.html#memoryview) — Official documentation for memoryview objects and the buffer protocol
- [struct — Interpret bytes as packed binary data](https://docs.python.org/3.14/library/struct.html) — Official documentation for the struct module including format characters and byte order

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*