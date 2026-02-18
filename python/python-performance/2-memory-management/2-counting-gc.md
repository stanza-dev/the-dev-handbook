---
source_course: "python-performance"
source_lesson: "python-performance-ref-counting-gc"
---

# Memory Management Internals

## Reference Counting

Python's primary memory management tracks references:

```python
import sys

x = [1, 2, 3]
print(sys.getrefcount(x))  # 2 (x + getrefcount arg)

y = x
print(sys.getrefcount(x))  # 3

del y
print(sys.getrefcount(x))  # 2

# When refcount reaches 0, memory is freed immediately
```

## The Cycle Problem

```python
# Reference counting fails here!
class Node:
    def __init__(self):
        self.next = None

a = Node()
b = Node()
a.next = b
b.next = a  # Cycle!

del a, b
# Refcounts are still 1 - memory leak!
```

## Generational Garbage Collector

Python's cyclic GC:
- **Generation 0**: New objects, collected frequently
- **Generation 1**: Survived one collection
- **Generation 2**: Long-lived objects, collected rarely

```python
import gc

# GC information
print(gc.get_count())      # Objects in each generation
print(gc.get_threshold())  # Collection thresholds

# Force collection
gc.collect()  # Run full GC
gc.collect(0)  # Run generation 0 only
```

## GC Control

```python
import gc

# Disable GC (careful!)
gc.disable()

# Run critical code without GC pauses
process_data()

# Re-enable and collect
gc.enable()
gc.collect()
```

## Code Examples

**Detecting reference cycles**

```python
import gc
import sys

# Detect reference cycles
gc.set_debug(gc.DEBUG_SAVEALL)

class Cyclic:
    def __init__(self):
        self.ref = None

a = Cyclic()
b = Cyclic()
a.ref = b
b.ref = a

del a, b
gc.collect()

# gc.garbage contains uncollectable objects
print(f"Garbage: {len(gc.garbage)}")
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*