---
source_course: "python-performance"
source_lesson: "python-performance-adaptive-interpreter"
---

# Tier 1: Adaptive Interpreter

Python 3.11 introduced specializing/adaptive bytecode.

## How It Works

1. Code starts with generic bytecodes
2. After execution, Python observes types
3. Bytecodes "specialize" based on observed types
4. If types change, bytecodes can "de-specialize"

## Specialization Examples

```
Generic:              Specialized:
BINARY_OP       →     BINARY_OP_ADD_INT
LOAD_ATTR       →     LOAD_ATTR_INSTANCE_VALUE  
LOAD_GLOBAL     →     LOAD_GLOBAL_MODULE
CALL            →     CALL_PY_EXACT_ARGS
```

## Why Specialization Helps

```python
def add_numbers(a, b):
    return a + b

# First calls: Generic BINARY_OP (slow)
# After seeing ints: BINARY_OP_ADD_INT (fast)

# If you call with strings:
add_numbers("hello", "world")
# De-specializes back to generic
```

## Quickening

Bytecodes are "quickened" after they've been executed:

```python
import sys

def hot_function():
    pass

# Quickened after ~8 calls
for _ in range(10):
    hot_function()

# Check specialization stats (3.11+)
if hasattr(sys, '_stats'):
    print(sys._stats())
```

## Code Examples

**Adaptive bytecode**

```python
# Viewing specialized bytecode
import dis

def typed_add(a: int, b: int) -> int:
    return a + b

# Call to trigger specialization
for _ in range(100):
    typed_add(1, 2)

# View specialized bytecode
dis.dis(typed_add, adaptive=True)
# Shows BINARY_OP_ADD_INT instead of BINARY_OP
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*