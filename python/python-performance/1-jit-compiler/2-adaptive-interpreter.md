---
source_course: "python-performance"
source_lesson: "python-performance-adaptive-interpreter"
---

# The Adaptive Interpreter (3.11+)

## Introduction

Python 3.11 introduced the specializing adaptive interpreter (PEP 659), the single biggest performance improvement in CPython's history. Instead of always executing generic bytecode, the interpreter observes the types flowing through your code and replaces generic instructions with type-specific fast paths. This lesson explains how specialization and de-specialization work under the hood.

## Key Concepts

- **Specialization**: The process of replacing a generic bytecode instruction with a type-specific variant after observing the actual types at runtime.
- **Quickening**: The mechanism that triggers specialization. After a bytecode has been executed approximately 8 times, CPython attempts to specialize it.
- **De-specialization**: When a specialized instruction encounters a type it was not optimized for, it reverts to the generic version and may re-specialize later.
- **Tier 1**: The name for the adaptive interpreter level of optimization. Tier 0 is the unspecialized generic interpreter; Tier 2 is the JIT compiler.
- **Adaptive bytecode**: Bytecodes that carry an internal counter tracking how many times they have been executed, enabling the quickening process.

## Real World Context

If you write a function that always adds integers, the adaptive interpreter will specialize `BINARY_OP` to `BINARY_OP_ADD_INT`, which skips type-checking overhead and calls the integer addition directly. This is why type-stable code in Python 3.11+ can be 10-60% faster than the same code in Python 3.10.

## Deep Dive

### How Specialization Works

When Python first executes a function, all bytecodes are generic. After about 8 executions of each instruction, CPython "quickens" it:

1. CPython observes the types of operands at each instruction
2. If a pattern is consistent (e.g., both operands are `int`), it replaces the instruction with a specialized variant
3. The specialized instruction skips generic type dispatch and runs a fast path

Here are the key specialization families:

```
Generic Instruction       Specialized Variant
─────────────────────     ─────────────────────────────────
BINARY_OP            →    BINARY_OP_ADD_INT
                          BINARY_OP_ADD_FLOAT
                          BINARY_OP_MULTIPLY_INT
LOAD_ATTR            →    LOAD_ATTR_INSTANCE_VALUE
                          LOAD_ATTR_MODULE
                          LOAD_ATTR_SLOT
LOAD_GLOBAL          →    LOAD_GLOBAL_MODULE
                          LOAD_GLOBAL_BUILTIN
CALL                 →    CALL_PY_EXACT_ARGS
                          CALL_BUILTIN_FAST
STORE_ATTR           →    STORE_ATTR_INSTANCE_VALUE
                          STORE_ATTR_SLOT
```

### Viewing Specialized Bytecode

You can see the specialized instructions using `dis.dis()` with `adaptive=True` after warming up the function:

```python
import dis

def multiply_ints(a, b):
    return a * b

# Warm up: call enough times to trigger specialization
for _ in range(100):
    multiply_ints(3, 7)

# View the specialized bytecode
dis.dis(multiply_ints, adaptive=True)
# You will see BINARY_OP_MULTIPLY_INT
# instead of the generic BINARY_OP
```

### De-specialization

If a specialized instruction encounters a type it was not designed for, it de-specializes back to generic and may try to re-specialize later:

```python
def add_values(a, b):
    return a + b

# Phase 1: Integers → specializes to BINARY_OP_ADD_INT
for _ in range(100):
    add_values(1, 2)

# Phase 2: Strings → de-specializes back to BINARY_OP
add_values("hello", " world")

# Phase 3: If strings continue, may re-specialize
# to BINARY_OP_ADD_UNICODE
for _ in range(100):
    add_values("hello", " world")
```

### Checking Specialization Statistics

CPython can report detailed specialization statistics when built with the `--enable-pystats` flag:

```python
import sys

# Only available in debug/stats builds
if hasattr(sys, '_stats'):
    stats = sys._stats()
    print(stats)
```

In Python 3.14, the specializing adaptive interpreter is also enabled in free-threaded mode (PEP 703), where it was previously disabled. This means free-threaded builds now benefit from the same specialization optimizations.

## Common Pitfalls

- **Mixing types defeats specialization**: A function that receives `int` on one call and `str` on the next will repeatedly specialize and de-specialize, potentially performing worse than a consistently generic function.
- **Assuming type hints trigger specialization**: Type annotations have zero effect on specialization. CPython specializes based on observed runtime types, not annotations. Annotations are for static type checkers like mypy.
- **Forgetting warmup in benchmarks**: Specialization requires several executions. If you benchmark a function on its first call, you are measuring unspecialized performance.

## Best Practices

- **Keep hot functions type-stable**: Pass the same types consistently to performance-critical functions so the adaptive interpreter can specialize effectively.
- **Use `dis.dis(func, adaptive=True)` to verify specialization**: After warming up a function, inspect its bytecode to confirm that specialization is happening where you expect.
- **Avoid polymorphic hot paths**: If a function must handle multiple types, consider splitting it into type-specific helpers so each one can specialize independently.

## Summary

- The adaptive interpreter (PEP 659, Python 3.11+) observes runtime types and replaces generic bytecodes with specialized fast-path variants
- Quickening triggers after approximately 8 executions of each instruction
- Key specializations include `BINARY_OP_ADD_INT`, `LOAD_ATTR_INSTANCE_VALUE`, and `CALL_PY_EXACT_ARGS`
- De-specialization occurs when a specialized instruction encounters an unexpected type, and re-specialization may follow
- In Python 3.14, the adaptive interpreter is now enabled in free-threaded mode as well

## Code Examples

**Warming up a function with consistent float types and viewing the specialized bytecode to verify that BINARY_OP has been replaced with BINARY_OP_MULTIPLY_FLOAT**

```python
import dis

def multiply_prices(prices, tax_rate):
    """Calculate total with tax for a list of prices."""
    total = 0.0
    for price in prices:
        total += price * tax_rate
    return total

# Warm up with consistent types to trigger specialization
test_prices = [19.99, 24.50, 9.99, 14.75]
for _ in range(100):
    multiply_prices(test_prices, 1.08)

# View specialized bytecode — BINARY_OP_MULTIPLY_FLOAT
# and BINARY_OP_ADD_FLOAT should appear
print("Specialized bytecode after warmup:")
dis.dis(multiply_prices, adaptive=True)
```


## Resources

- [PEP 659 — Specializing Adaptive Interpreter](https://peps.python.org/pep-0659/) — The PEP describing the design and rationale of Python's specializing adaptive interpreter
- [What's New in Python 3.11 — Performance](https://docs.python.org/3.14/whatsnew/3.11.html#faster-cpython) — Official documentation of the Faster CPython performance improvements in 3.11

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*