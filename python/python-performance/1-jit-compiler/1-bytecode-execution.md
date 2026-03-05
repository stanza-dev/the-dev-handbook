---
source_course: "python-performance"
source_lesson: "python-performance-bytecode-execution"
---

# Python Bytecode & Execution

## Introduction

Python is often called an "interpreted" language, but that is only half the story. CPython actually compiles your source code to bytecode before executing it, and understanding this compilation pipeline is the first step to writing faster Python. In this lesson, you will learn how Python transforms source code into bytecode instructions and how the interpreter evaluates them.

## Key Concepts

- **Bytecode**: The low-level, platform-independent instruction set that CPython executes. Stored in `.pyc` files under `__pycache__/`.
- **Abstract Syntax Tree (AST)**: The tree representation of your source code that the compiler converts to bytecode.
- **Eval Loop**: The central `while` loop in CPython (`_PyEval_EvalFrameDefault`) that fetches, decodes, and executes bytecode instructions one at a time.
- **`dis` module**: Python's built-in bytecode disassembler that lets you inspect the instructions generated for any function.
- **BINARY_OP**: The unified arithmetic bytecode instruction introduced in Python 3.12, replacing the older per-operation opcodes like `BINARY_ADD`.

## Real World Context

When you profile a slow function and see that its pure-Python loop is the bottleneck, understanding bytecode tells you exactly what the interpreter is doing on every iteration. This knowledge helps you decide whether to restructure the loop, push work into a C extension, or rely on the JIT compiler introduced in Python 3.13+.

## Deep Dive

### The Compilation Pipeline

Every time Python runs your code, it goes through these stages:

1. **Source Code** is parsed into an **AST**
2. **AST** is compiled into **Bytecode** (`.pyc` files)
3. **Bytecode** is executed by the **Interpreter** (eval loop)

You can inspect the bytecode for any function using the `dis` module. Here is what the output looks like in Python 3.14:

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```

Python 3.14 output:

```
  1           RESUME                   0

  2           LOAD_FAST                0 (a)
              LOAD_FAST                1 (b)
              BINARY_OP                0 (+)
              RETURN_VALUE
```

Notice that the instruction is `BINARY_OP` with an argument indicating the operator (`+`), not the older `BINARY_ADD` opcode that existed before Python 3.12. The `RESUME` instruction at the top is another modern addition that handles generator/coroutine bookkeeping.

### The Eval Loop

CPython's eval loop is essentially a giant switch statement in C. On every iteration it:

1. Fetches the next bytecode instruction
2. Decodes the opcode and its argument
3. Dispatches to the handler for that opcode
4. Repeats

This dispatch overhead is one reason pure Python is slower than compiled languages. Each operation (even `a + b`) requires fetching an instruction, looking up types, calling the appropriate C function, and handling potential errors.

### Inspecting Code Objects

Every function has a `__code__` object that holds its bytecode and metadata:

```python
def compute(x, y):
    total = x + y
    return total * 2

code = compute.__code__
print(f"Stack size: {code.co_stacksize}")
print(f"Constants: {code.co_consts}")
print(f"Local variables: {code.co_varnames}")
print(f"Bytecode bytes: {code.co_code.hex()}")
```

### The Optimization Journey

Python's execution has been getting faster with each release:

| Python Version | Optimization |
|---|---|
| 3.11 | Adaptive Interpreter (Tier 1 specialization) |
| 3.12 | BINARY_OP replaces BINARY_ADD/BINARY_MULTIPLY/etc. |
| 3.13 | Experimental JIT compiler (Tier 2, PEP 744) |
| 3.14 | Improved JIT + Tail-Call Interpreter (3-5% faster) |

The tail-call interpreter in Python 3.14 restructures the eval loop so that each opcode handler directly tail-calls the next one, eliminating the central dispatch overhead and yielding a 3-5% improvement on the pyperformance benchmark suite.

## Common Pitfalls

- **Assuming `BINARY_ADD` still exists**: If you are reading older Python books or tutorials, they will show `BINARY_ADD` in `dis` output. Since Python 3.12, this has been replaced by `BINARY_OP`. Always check which Python version the material targets.
- **Ignoring the `RESUME` instruction**: Python 3.14 `dis` output starts with `RESUME`. This is not a bug in your code; it is a required instruction for generator and coroutine support and appears in every function.
- **Over-optimizing bytecode manually**: Understanding bytecode is useful for diagnosis, but you should almost never try to hand-craft bytecode. The compiler and adaptive interpreter handle optimization for you.

## Best Practices

- **Use `dis.dis()` to understand performance-critical code**: When a function is slow, disassembling it reveals exactly what operations the interpreter runs, helping you identify unnecessary work.
- **Profile before you optimize**: Bytecode knowledge is powerful but should be combined with profiling (`cProfile`, `timeit`) to find actual bottlenecks rather than guessing.
- **Keep your Python version up to date**: Each release brings interpreter improvements. The jump from 3.10 to 3.14 includes specialization, JIT compilation, and the tail-call interpreter.

## Summary

- CPython compiles source code to bytecode, then executes it in an eval loop that fetches and dispatches instructions one at a time
- The `dis` module disassembles functions into human-readable bytecode; in Python 3.14 you will see `RESUME` and `BINARY_OP` (not `BINARY_ADD`)
- Every function has a `__code__` object containing its bytecode, constants, and metadata
- Python 3.14 introduces a tail-call interpreter that eliminates central dispatch overhead for a 3-5% speed improvement
- Understanding the bytecode pipeline prepares you for the adaptive interpreter and JIT compiler covered in the next lessons

## Code Examples

**Disassembling a function with dis.dis() to inspect Python 3.14 bytecode instructions and examining the code object metadata**

```python
import dis
import sys

def add(a, b):
    return a + b

# Disassemble to see Python 3.14 bytecode
# Shows RESUME and BINARY_OP (not the old BINARY_ADD)
dis.dis(add)

# Inspect the underlying code object
code = add.__code__
print(f"\nCode object details:")
print(f"  Stack size: {code.co_stacksize}")
print(f"  Constants: {code.co_consts}")
print(f"  Local vars: {code.co_varnames}")
print(f"  Python version: {sys.version}")
```


## Resources

- [dis — Disassembler for Python bytecode](https://docs.python.org/3.14/library/dis.html) — Official documentation for the dis module including all bytecode instructions
- [Python bytecode instructions](https://docs.python.org/3.14/library/dis.html#python-bytecode-instructions) — Complete reference for all Python 3.14 bytecode opcodes including BINARY_OP and RESUME

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*