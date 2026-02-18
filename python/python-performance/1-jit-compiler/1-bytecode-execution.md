---
source_course: "python-performance"
source_lesson: "python-performance-bytecode-execution"
---

# Understanding Python Execution

## How Python Runs Code

1. **Source Code** → Parser → **AST** (Abstract Syntax Tree)
2. **AST** → Compiler → **Bytecode** (.pyc files)
3. **Bytecode** → Interpreter → **Execution**

## Viewing Bytecode

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
# Output:
#   2           0 LOAD_FAST                0 (a)
#               2 LOAD_FAST                1 (b)
#               4 BINARY_ADD
#               6 RETURN_VALUE
```

## The Interpreter Loop

CPython uses a giant switch statement (the "eval loop") that:
1. Fetches the next bytecode instruction
2. Executes it
3. Repeats

## Why Python is "Slow"

- **Dynamic typing**: Type checks at every operation
- **Object overhead**: Everything is an object
- **Interpretation**: No machine code optimization
- **GIL**: Single-threaded execution

## The Optimization Journey

| Python Version | Optimization |
|----------------|-------------|
| 3.11 | Adaptive Interpreter (Tier 1) |
| 3.13 | Experimental JIT (Tier 2) |
| 3.14 | Improved JIT |
| 3.15 | Mature JIT |

## Code Examples

**Inspecting bytecode**

```python
import dis
import sys

def example():
    x = 1
    y = 2
    return x + y

# View bytecode
dis.dis(example)

# Get bytecode object
code = example.__code__
print(f"Stack size: {code.co_stacksize}")
print(f"Constants: {code.co_consts}")
print(f"Local vars: {code.co_varnames}")
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*