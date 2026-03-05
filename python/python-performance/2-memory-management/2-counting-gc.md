---
source_course: "python-performance"
source_lesson: "python-performance-ref-counting-gc"
---

# Reference Counting & Incremental GC

## Introduction

Python uses two complementary mechanisms to manage memory: reference counting as the primary system and a cyclic garbage collector as a backup. In Python 3.14, the garbage collector has been fundamentally redesigned to be incremental, dramatically reducing pause times. This lesson covers both systems in detail with accurate Python 3.14 information.

## Key Concepts

- **Reference Counting**: The primary memory management mechanism. Every object maintains a count of how many references point to it, and is immediately freed when the count reaches zero.
- **Cyclic Garbage Collector**: A secondary mechanism that detects and collects reference cycles (groups of objects that reference each other) which reference counting cannot handle.
- **Incremental GC**: New in Python 3.12+, the garbage collector now uses an incremental algorithm with 2 internal generations (young and old) instead of the old 3-generation non-incremental approach.
- **GC Pause Time**: The time the garbage collector stops the program to trace and collect objects. The incremental design reduces maximum pause times by orders of magnitude.
- **Reference Cycle**: A group of objects that reference each other in a loop, preventing their reference counts from ever reaching zero.

## Real World Context

In a web server handling thousands of requests per second, a non-incremental GC pause of 50-100ms can cause visible latency spikes. Python 3.14's incremental GC keeps individual pause times tiny and predictable, making Python much more suitable for latency-sensitive applications.

## Deep Dive

### Reference Counting in Action

Python's primary memory management is reference counting. Each object tracks how many references point to it:

```python
import sys

data = [1, 2, 3]
print(sys.getrefcount(data))  # 2 (data + getrefcount argument)

alias = data
print(sys.getrefcount(data))  # 3 (data + alias + getrefcount)

del alias
print(sys.getrefcount(data))  # 2 again

# When refcount reaches 0, memory is freed immediately
# No GC needed for non-cyclic objects
```

The key advantage of reference counting is **deterministic deallocation**: objects are freed the instant their last reference disappears, without waiting for a garbage collection cycle.

### The Reference Cycle Problem

Reference counting fails when objects reference each other in a cycle:

```python
class Node:
    def __init__(self, name):
        self.name = name
        self.neighbor = None

a = Node("A")
b = Node("B")
a.neighbor = b  # A → B
b.neighbor = a  # B → A (cycle!)

del a, b
# Both objects still have refcount 1 (from each other)
# Reference counting alone cannot free them
# The cyclic GC must detect and break the cycle
```

### Python 3.14: Incremental Garbage Collection

Python 3.14's garbage collector uses an **incremental** algorithm that is fundamentally different from the old design:

**Old design (Python 3.11 and earlier):**
- 3 generations (gen 0, gen 1, gen 2)
- Full generation 2 collections could pause the program for a long time
- Pause times proportional to total number of objects in a generation

**New design (Python 3.12+ / 3.14):**
- 2 internal generations: **young** and **old**
- The old generation is collected **incrementally** — a small portion at a time
- Maximum pause times reduced by **orders of magnitude**
- `gc.get_stats()` still returns 3 dicts for backward compatibility

```python
import gc

# gc.collect() behavior in Python 3.14:
gc.collect(0)  # Collect young generation
gc.collect(1)  # Collect young + an increment of old generation
gc.collect(2)  # Full collection (young + all of old)

# Thresholds and stats
print(f"Thresholds: {gc.get_threshold()}")
print(f"Stats: {gc.get_stats()}")  # Still 3 dicts for compat
print(f"Object counts: {gc.get_count()}")
```

### How Incremental Collection Works

Instead of scanning all old-generation objects at once, the incremental GC:

1. Maintains a **young** generation for newly created objects
2. Promotes surviving young objects to the **old** generation
3. Scans the old generation in **small increments** on each collection cycle
4. Spreads the work of a full collection across many small pauses

This means a program with millions of long-lived objects no longer experiences the occasional long pause that occurred when the old generation 2 was fully collected.

### Controlling the GC

You can tune or temporarily disable the garbage collector:

```python
import gc

# Disable GC for latency-critical section
gc.disable()
try:
    process_real_time_data()  # No GC pauses during this
finally:
    gc.enable()
    gc.collect()  # Clean up after

# Tune collection thresholds
# Default: (700, 10, 10)
gc.set_threshold(1000, 15, 15)  # Less frequent collection

# Debug: find reference cycles
gc.set_debug(gc.DEBUG_SAVEALL)
```

### Breaking Cycles Explicitly

You can help reference counting work by breaking cycles yourself:

```python
class TreeNode:
    def __init__(self, value):
        self.value = value
        self.parent = None
        self.children = []

    def add_child(self, child):
        self.children.append(child)
        child.parent = self  # Creates parent ↔ child cycle

    def detach(self):
        """Break the cycle explicitly."""
        if self.parent:
            self.parent.children.remove(self)
            self.parent = None

# Explicit cleanup is faster than waiting for GC
node = TreeNode("leaf")
node.detach()  # Reference counting can now free it immediately
```

## Common Pitfalls

- **Assuming the old 3-generation model**: Many tutorials and books describe Python's GC as having 3 generations. Since Python 3.12, the GC internally uses 2 generations (young + old) with incremental collection of the old generation. `gc.get_stats()` still returns 3 dicts for backward compatibility, which adds to the confusion.
- **Disabling GC without re-enabling it**: If you disable the GC for a performance-critical section, always re-enable it in a `finally` block. Forgetting this will cause memory leaks from uncollected reference cycles.
- **Over-relying on `__del__` for cleanup**: The `__del__` finalizer is called when an object is being deallocated, but the timing is unpredictable for objects in reference cycles. Use context managers (`with` statements) for reliable cleanup.

## Best Practices

- **Prefer weak references for caches and observers**: Use `weakref.ref()` or `weakref.WeakValueDictionary` to avoid creating reference cycles in cache and observer patterns.
- **Use context managers for resource cleanup**: Instead of relying on `__del__`, use `with` statements to ensure files, connections, and locks are released deterministically.
- **Monitor GC activity in production**: Use `gc.get_stats()` and `gc.callbacks` to track GC frequency and pause times in your application.

## Summary

- Reference counting is Python's primary memory management mechanism, freeing objects immediately when their reference count reaches zero
- The cyclic garbage collector handles reference cycles that reference counting cannot detect
- Python 3.14's GC uses an incremental algorithm with 2 internal generations (young + old), reducing maximum pause times by orders of magnitude compared to the old 3-generation design
- `gc.collect(0)` collects young, `gc.collect(1)` collects young + increment of old, `gc.collect(2)` does a full collection
- `gc.get_stats()` still returns 3 dicts for backward compatibility even though the internal model has changed

## Code Examples

**Demonstrating reference counting behavior with sys.getrefcount(), inspecting the incremental GC configuration, and creating a reference cycle that requires garbage collection**

```python
import gc
import sys

# Demonstrate reference counting
data = {"key": "value"}
print(f"Refcount after creation: {sys.getrefcount(data)}")

alias = data
print(f"Refcount after alias:    {sys.getrefcount(data)}")

del alias
print(f"Refcount after del:      {sys.getrefcount(data)}")

# Demonstrate the incremental GC in Python 3.14
print(f"\nGC thresholds: {gc.get_threshold()}")
print(f"GC stats (3 dicts for compat): {gc.get_stats()}")
print(f"GC object counts: {gc.get_count()}")

# Create and detect a reference cycle
class CyclicNode:
    def __init__(self):
        self.ref = None

a = CyclicNode()
b = CyclicNode()
a.ref = b
b.ref = a  # Cycle: a → b → a

del a, b
collected = gc.collect()  # Needed to free cyclic objects
print(f"\nCollected {collected} cyclic objects")
```


## Resources

- [gc — Garbage Collector interface](https://docs.python.org/3.14/library/gc.html) — Official documentation for the gc module including collection, thresholds, and debugging
- [What's New in Python 3.14 — Incremental GC](https://docs.python.org/3.14/whatsnew/3.14.html) — Release notes describing the incremental garbage collector improvements in Python 3.14

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*