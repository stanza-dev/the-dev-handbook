---
source_course: "python-concurrency"
source_lesson: "python-concurrency-shared-memory"
---

# High-Performance Shared Memory

`shared_memory` provides direct memory sharing without copying.

## Creating Shared Memory

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

## Sharing NumPy Arrays

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

## ShareableList

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

## Memory Management

```python
import atexit

# Ensure cleanup on exit
shm = shared_memory.SharedMemory(create=True, size=1000)
atexit.register(shm.unlink)
atexit.register(shm.close)
```

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*