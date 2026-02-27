---
source_course: "python-concurrency"
source_lesson: "python-concurrency-shared-memory"
---

# Shared Memory (3.8+)

## Introduction
While queues and pipes copy data between processes (incurring serialization overhead), `shared_memory` provides direct memory sharing without copying. This is essential for high-performance workloads like sharing large NumPy arrays between processes. This lesson covers creating shared memory blocks, sharing NumPy arrays, using `ShareableList`, and the critical importance of proper cleanup.

## Key Concepts
- **SharedMemory**: An OS-level shared memory block that can be accessed by multiple processes without copying data.
- **Buffer protocol**: SharedMemory exposes a `buf` attribute that supports Python's buffer protocol, allowing zero-copy access from NumPy and other libraries.
- **ShareableList**: A fixed-size list stored in shared memory that supports basic types (int, float, str, bytes, None).
- **unlink()**: Releases the shared memory block from the OS. Forgetting to call it leaks memory that persists until reboot.

## Real World Context
A multi-process image pipeline loads a 2GB dataset of images into a NumPy array. Without shared memory, each of the 8 worker processes would need its own copy, requiring 16GB of RAM. With `SharedMemory`, all processes access the same memory block, keeping total usage at 2GB. The workers modify their assigned regions in place with no serialization overhead.

## Deep Dive

### Creating Shared Memory

```python
from multiprocessing import shared_memory
import numpy as np

# Create shared memory block
shm = shared_memory.SharedMemory(create=True, size=1000)

# Access as buffer
buffer = shm.buf
buffer[0] = 255

# Clean up (important!)
shm.close()
shm.unlink()  # Remove from system
```

### Sharing NumPy Arrays

```python
from multiprocessing import shared_memory
import numpy as np

# Create array in shared memory
arr = np.array([1, 2, 3, 4, 5])
shm = shared_memory.SharedMemory(create=True, size=arr.nbytes)
shared_arr = np.ndarray(arr.shape, dtype=arr.dtype, buffer=shm.buf)
shared_arr[:] = arr  # Copy data to shared memory

# In another process, attach to existing shared memory
shm2 = shared_memory.SharedMemory(name=shm.name)
shared_arr2 = np.ndarray(arr.shape, dtype=arr.dtype, buffer=shm2.buf)
# shared_arr2 now sees the same data!
```

### ShareableList

```python
from multiprocessing.shared_memory import ShareableList

# Create shareable list
sl = ShareableList([1, 2, 3, 4, 5])

# Access by name from another process
sl2 = ShareableList(name=sl.shm.name)
sl2[0] = 99  # Visible to all processes

# Clean up
sl.shm.close()
sl.shm.unlink()
```

### Memory Management

```python
import atexit

# Ensure cleanup on exit
shm = shared_memory.SharedMemory(create=True, size=1000)
atexit.register(shm.unlink)
atexit.register(shm.close)
```

## Common Pitfalls
1. **Forgetting to call `unlink()`** — Shared memory blocks persist at the OS level. If you only call `close()`, the block remains allocated until the system reboots. Always call `unlink()` from exactly one process (the creator).
2. **Calling `unlink()` from every process** — Only the process that created the shared memory should `unlink()` it. Other processes should only call `close()`. Unlinking from a process that is still using it causes segfaults.
3. **Not registering cleanup with `atexit`** — If your program crashes before cleanup, the shared memory leaks. Register `shm.close()` and `shm.unlink()` with `atexit` as a safety net.

## Best Practices
1. **Use `atexit.register()` for cleanup safety** — Register `close` and `unlink` immediately after creating shared memory so it is cleaned up even if the program exits unexpectedly.
2. **Pass the `shm.name` string to child processes** — Instead of passing the SharedMemory object, pass its name string. Child processes attach to the existing block by name, which works across all start methods.

## Summary
- `shared_memory.SharedMemory` provides zero-copy data sharing between processes, ideal for large arrays.
- NumPy arrays can be mapped directly onto shared memory buffers for high-performance parallel processing.
- `ShareableList` stores fixed-size lists of basic types in shared memory.
- Always call `close()` in every process and `unlink()` in exactly one process (the creator).
- Use `atexit.register()` to prevent shared memory leaks from crashes.

## Code Examples

**Sharing NumPy arrays**

```python
from multiprocessing import Process, shared_memory
import numpy as np

def worker(shm_name, shape, dtype):
    # Attach to existing shared memory
    shm = shared_memory.SharedMemory(name=shm_name)
    arr = np.ndarray(shape, dtype=dtype, buffer=shm.buf)
    
    # Modify in place
    arr *= 2
    
    shm.close()

if __name__ == '__main__':
    # Create shared array
    original = np.array([1, 2, 3, 4, 5], dtype=np.int64)
    shm = shared_memory.SharedMemory(create=True, size=original.nbytes)
    shared = np.ndarray(original.shape, dtype=original.dtype, buffer=shm.buf)
    shared[:] = original
    
    # Modify in subprocess
    p = Process(target=worker, args=(shm.name, shared.shape, shared.dtype))
    p.start()
    p.join()
    
    print(shared)  # [2, 4, 6, 8, 10]
    
    shm.close()
    shm.unlink()
```


## Resources

- [Shared Memory](https://docs.python.org/3.14/library/multiprocessing.shared_memory.html) — High-performance shared memory for direct data sharing between processes

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*