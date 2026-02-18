---
source_course: "python-performance"
source_lesson: "python-performance-buffer-protocol"
---

# Zero-Copy Memory Access

The buffer protocol allows direct memory access without copying.

## memoryview

```python
data = bytearray(b"Hello World")
view = memoryview(data)

# View into same memory
view[0] = ord('h')
print(data)  # b'hello World'

# Slicing without copying
subview = view[6:]  # Still same memory!
subview[0] = ord('w')
print(data)  # b'hello world'
```

## struct Module

```python
import struct

# Pack data into bytes
packed = struct.pack('iif', 1, 2, 3.14)
print(packed)  # Binary data

# Unpack bytes into values
values = struct.unpack('iif', packed)
print(values)  # (1, 2, 3.14)

# Efficient for network/file protocols
header = struct.pack('>BHI', 1, 256, 65536)  # Big-endian
```

## array Module

```python
import array

# Typed array (like C arrays)
arr = array.array('i', [1, 2, 3, 4, 5])  # Integers

# Much more memory-efficient than list
print(arr.itemsize)  # Bytes per item
print(arr.buffer_info())  # (address, length)

# Supports buffer protocol
view = memoryview(arr)
```

## NumPy Integration

```python
import numpy as np

# Create array from buffer
data = bytearray(16)
np_arr = np.frombuffer(data, dtype=np.float32)

# Shares memory!
np_arr[0] = 3.14
print(data[:4])  # Modified
```

## Code Examples

**Binary data parsing**

```python
import struct

# Binary file parsing example
def parse_header(data):
    # Header: magic (4 bytes), version (2 bytes), size (4 bytes)
    header_format = '<4sHI'  # Little-endian
    
    magic, version, size = struct.unpack_from(header_format, data)
    return {
        'magic': magic,
        'version': version,
        'size': size
    }

# Create test header
header = struct.pack('<4sHI', b'DATA', 1, 1024)
print(parse_header(header))
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*