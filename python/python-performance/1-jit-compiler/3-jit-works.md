---
source_course: "python-performance"
source_lesson: "python-performance-how-jit-works"
---

# Tier 2: The JIT Compiler

Python 3.13+ includes a JIT that compiles hot code to machine code.

## The JIT Pipeline

1. **Detect Hot Code**: Track execution frequency
2. **Trace Formation**: Record sequence of operations
3. **Optimization**: Remove redundant checks
4. **Code Generation**: Emit machine code

## Copy-and-Patch Architecture

Unlike traditional JITs (LLVM-based), CPython uses "Copy-and-Patch":

```
Pre-compiled templates:
[Machine code for ADD_INT with holes]
[Machine code for LOAD_FAST with holes]

At runtime:
1. Copy template
2. Patch holes with actual values
3. Execute
```

**Benefits:**
- Very fast compilation
- Low memory overhead
- Simple implementation

## Micro-ops (uops)

The JIT works on a lower-level representation:

```python
# Python bytecode:
BINARY_OP ADD

# Becomes uops:
_GUARD_TYPE_INT(left)
_GUARD_TYPE_INT(right)
_BINARY_OP_ADD_INT
```

## Checking JIT Status

```python
import sys

# Check if JIT is enabled
if hasattr(sys, '_jit'):
    print(f"JIT enabled: {sys._jit.is_enabled()}")
```

## Code Examples

**JIT-friendly patterns**

```python
# JIT-friendly code patterns

# Good: Consistent types
def sum_ints(n: int) -> int:
    total = 0
    for i in range(n):
        total += i  # Always int + int
    return total

# Bad: Type instability
def sum_mixed(items):
    total = 0
    for item in items:
        total += item  # Could be int, float, str...
    return total

# The JIT optimizes sum_ints much better!
```


## Resources

- [PEP 744](https://peps.python.org/pep-0744/) — JIT Compilation

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*