---
source_course: "python-concurrency"
source_lesson: "python-concurrency-migrating-nogil"
---

# Migrating to Free-Threaded Python

## Introduction
Migrating an existing codebase to free-threaded Python requires a systematic approach: audit shared mutable state, add synchronization, verify library compatibility, and test under stress. This lesson provides a step-by-step migration guide with concrete code patterns for each phase.

## Key Concepts
- **Code audit**: The process of scanning your codebase for patterns that are unsafe without the GIL, such as global mutable state and lazy initialization.
- **Double-checked locking**: A pattern where you check a condition, acquire a lock, then check again to avoid unnecessary locking on the fast path.
- **Thread sanitizer**: A tool that detects data races at runtime by monitoring memory accesses across threads.
- **Conditional execution**: Writing code that behaves differently depending on whether the GIL is enabled or not.

## Real World Context
A company wants to migrate their Django application to free-threaded Python for better CPU-bound request handling. Their codebase has module-level caches, global counters, and several singleton patterns. Without a systematic migration, they would encounter intermittent data corruption in production. Following the audit-synchronize-test methodology prevents these issues.

## Deep Dive

### Step 1: Audit Your Code

```python
# Look for these patterns:

# 1. Global mutable state
global_cache = {}  # Needs protection

# 2. Module-level mutables
instance_count = 0  # Race condition

# 3. Lazy initialization
_singleton = None
def get_singleton():
    global _singleton
    if _singleton is None:  # Check-then-act
        _singleton = Expensive()
    return _singleton
```

### Step 2: Add Synchronization

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

### Step 3: Test Thoroughly

```bash
# Run tests with thread sanitizer
python -X faulthandler your_tests.py

# Stress test with many threads
python -m pytest --parallel-threads=16
```

### Library Compatibility

```python
# Check if library is free-threading compatible
import some_library

# Look for these indicators:
# - Uses proper locking internally
# - Declares thread-safety in docs
# - Tested with free-threaded Python
```

### Conditional Code

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

## Common Pitfalls
1. **Skipping the audit and jumping to testing** — Race conditions are often intermittent. Tests may pass 99 times and fail on the 100th. A code audit catches issues systematically rather than relying on luck.
2. **Applying locks too broadly** — Wrapping entire functions in locks defeats the purpose of parallelism. Identify the exact shared state and lock only the minimum critical section.
3. **Forgetting about third-party libraries** — Your code may be thread-safe, but a dependency that uses global state internally can still cause data corruption.

## Best Practices
1. **Use double-checked locking for singletons** — The outer check avoids lock contention on the fast path; the inner check prevents duplicate initialization under contention.
2. **Run stress tests with high thread counts** — Race conditions hide at low concurrency. Test with 16+ threads to increase the probability of exposing them.
3. **Maintain a library compatibility checklist** — Track which of your dependencies have been verified for free-threading and plan upgrades for those that have not.

## Summary
- Migration follows three phases: audit shared mutable state, add synchronization, and test under stress.
- Common unsafe patterns include global mutable state, module-level counters, and lazy initialization without locking.
- Double-checked locking fixes singleton patterns with minimal performance overhead.
- Use `sys._is_gil_enabled()` to write code that adapts to both GIL and free-threaded builds.
- Always verify third-party library compatibility before migrating.

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


## Resources

- [Free-threading Extensions](https://docs.python.org/3.14/howto/free-threading-extensions.html) — Guide for migrating C extensions to free-threaded Python

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*