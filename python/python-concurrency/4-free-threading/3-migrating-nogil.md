---
source_course: "python-concurrency"
source_lesson: "python-concurrency-migrating-nogil"
---

# Migration Guide

## Step 1: Audit Your Code

```python
# Look for these patterns:

# 1. Global mutable state
global_cache = {}  # ⚠️ Needs protection

# 2. Module-level mutables
instance_count = 0  # ⚠️ Race condition

# 3. Lazy initialization
_singleton = None
def get_singleton():
    global _singleton
    if _singleton is None:  # ⚠️ Check-then-act
        _singleton = Expensive()
    return _singleton
```

## Step 2: Add Synchronization

```python
import threading

# Fix lazy initialization
_singleton_lock = threading.Lock()
_singleton = None

def get_singleton():
    global _singleton
    if _singleton is None:
        with _singleton_lock:
            if _singleton is None:  # Double-check
                _singleton = Expensive()
    return _singleton
```

## Step 3: Test Thoroughly

```bash
# Run tests with thread sanitizer
python -X faulthandler your_tests.py

# Stress test with many threads
python -m pytest --parallel-threads=16
```

## Library Compatibility

```python
# Check if library is free-threading compatible
import some_library

# Look for these indicators:
# - Uses proper locking internally
# - Declares thread-safety in docs
# - Tested with free-threaded Python
```

## Conditional Code

```python
import sys

def optimized_operation():
    if hasattr(sys, '_is_gil_enabled') and not sys._is_gil_enabled():
        # Use parallel approach
        return parallel_implementation()
    else:
        # Use serial approach
        return serial_implementation()
```

## Code Examples

**Thread-safe singleton**

```python
# Thread-safe singleton pattern
import threading

class Singleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                # Double-checked locking
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

# Usage
s1 = Singleton()
s2 = Singleton()
assert s1 is s2  # Same instance
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*