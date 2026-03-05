---
source_course: "python-performance"
source_lesson: "python-performance-how-jit-works"
---

# How the JIT Works

## Introduction

Python 3.13 introduced an experimental JIT compiler (PEP 744), and Python 3.14 significantly improves it alongside the new tail-call interpreter. The JIT uses a novel "copy-and-patch" architecture that is radically different from traditional JIT compilers found in Java or JavaScript engines. This lesson explains how hot code is detected, how traces are formed, and how machine code is generated.

## Key Concepts

- **Copy-and-Patch**: A code generation technique where pre-compiled machine code templates are copied and patched with runtime values, rather than generating machine code from scratch.
- **Micro-ops (uops)**: A lower-level intermediate representation that the JIT operates on. Each Python bytecode is decomposed into one or more micro-ops before optimization.
- **Trace Formation**: The process of recording a linear sequence of micro-ops from a hot code path, which becomes the input for JIT compilation.
- **Guard**: A runtime check inserted into JIT-compiled code that validates assumptions (e.g., "this variable is still an int"). If a guard fails, execution falls back to the interpreter.
- **Tail-Call Interpreter**: New in Python 3.14, this restructures the eval loop so each opcode handler tail-calls the next, yielding a 3-5% improvement on pyperformance benchmarks.

## Real World Context

Traditional JIT compilers (like V8 for JavaScript or HotSpot for Java) use LLVM or custom code generators that are complex and slow to compile. CPython's copy-and-patch approach trades peak optimization for compilation speed: the JIT compiles quickly with low memory overhead, which is critical because Python programs often have short runtimes where a slow JIT would never pay back its compilation cost.

## Deep Dive

### The JIT Pipeline

The optimization pipeline in Python 3.14 has distinct tiers:

```
Tier 0: Generic bytecode interpreter
   ↓ (after ~8 executions per instruction)
Tier 1: Adaptive/Specializing interpreter (PEP 659)
   ↓ (after detecting hot traces)
Tier 2: JIT-compiled machine code
```

When the JIT detects hot code:

1. **Trace Recording**: The interpreter records a sequence of micro-ops from a hot loop or frequently called code path
2. **Optimization**: Redundant guards are eliminated, constants are folded, and type checks are hoisted out of loops
3. **Code Generation**: Copy-and-patch produces machine code from the optimized micro-ops
4. **Execution**: Future iterations jump directly into the compiled machine code

### Copy-and-Patch in Detail

Traditional JITs build machine code instruction by instruction, which is complex and slow. Copy-and-patch takes a different approach:

```
At build time (when CPython is compiled):
  - Pre-compile machine code templates for each micro-op
  - Each template has "holes" where runtime values go

At runtime (when JIT compiles Python code):
  1. Copy the appropriate template into an executable buffer
  2. Patch the holes with actual runtime values
  3. Chain templates together to form the compiled trace
```

This results in very fast compilation (microseconds, not milliseconds) with reasonable code quality.

### Micro-ops

The JIT works on a lower-level representation than Python bytecodes. A single bytecode is decomposed into multiple micro-ops:

```python
# Python bytecode for: total += price
# BINARY_OP  0 (+)

# Becomes these micro-ops:
# _GUARD_TYPE_VERSION   (check object type hasn't changed)
# _GUARD_DORV_VALUES    (check dict version)
# _LOAD_ATTR_INSTANCE_VALUE
# _GUARD_TYPE_INT       (verify left operand is int)
# _GUARD_TYPE_INT       (verify right operand is int)
# _BINARY_OP_ADD_INT    (fast integer addition)
```

The optimizer can then remove redundant guards. For example, if the same variable is checked twice in a loop, the second check can be eliminated.

### The Tail-Call Interpreter

Python 3.14 introduces the tail-call interpreter, which restructures the eval loop. Instead of a central dispatch loop:

```c
// Old approach: central dispatch
while (1) {
    opcode = next_instruction();
    switch (opcode) {
        case LOAD_FAST: handle_load_fast(); break;
        case BINARY_OP: handle_binary_op(); break;
        // ...
    }
}
```

The tail-call interpreter makes each handler directly call the next:

```c
// New approach: tail-call dispatch
void handle_load_fast(...) {
    // do work
    DISPATCH();  // tail-call to next handler
}
```

This eliminates the overhead of returning to the central loop, resulting in a 3-5% improvement across the pyperformance benchmark suite.

### Checking JIT Availability

The JIT must be enabled at CPython build time. You can check if it is available:

```python
import sys

# Check for JIT support in the build
jit_available = hasattr(sys, '_jit')
print(f"JIT available: {jit_available}")

# The JIT is still experimental in 3.14
# Enable at build time with: --enable-experimental-jit
```

## Common Pitfalls

- **Expecting the JIT to be enabled by default**: In Python 3.14, the JIT is still experimental. You must build CPython with `--enable-experimental-jit` to use it. Standard Python distributions from python.org do not include it yet.
- **Assuming the JIT helps all code equally**: The JIT targets hot loops and frequently executed paths. One-shot initialization code or I/O-bound programs will see little to no benefit.
- **Confusing copy-and-patch with tracing JIT**: Copy-and-patch is the code generation technique (how machine code is produced). Trace formation is how the JIT decides what to compile. They are complementary but separate concepts.

## Best Practices

- **Write tight, type-stable loops**: The JIT benefits most from loops where types are predictable and operations are consistent, allowing guards to be optimized away.
- **Allow warmup time in benchmarks**: The JIT needs several iterations to detect hot code, form traces, and compile. Include a warmup phase before measuring performance.
- **Stay informed on JIT progress**: The JIT is rapidly evolving. Check the Python release notes for each version to understand what has improved.

## Summary

- Python 3.14's JIT uses copy-and-patch: pre-compiled machine code templates are copied and patched with runtime values for extremely fast compilation
- The JIT operates on micro-ops, a lower-level representation than bytecodes, enabling fine-grained optimization like guard elimination
- Trace formation records hot code paths as linear sequences of micro-ops that become the input for compilation
- The tail-call interpreter in 3.14 eliminates central dispatch overhead for a 3-5% improvement across benchmarks
- The JIT is still experimental in 3.14 and must be enabled at build time with `--enable-experimental-jit`

## Code Examples

**A JIT-friendly function with a tight, type-stable loop that benefits from both specialization and JIT compilation, with runtime JIT availability checking**

```python
import sys
import dis

def sum_squares(n):
    """Sum of squares — a JIT-friendly tight loop."""
    total = 0
    for i in range(n):
        total += i * i  # Type-stable: always int * int
    return total

# Warm up to trigger specialization (and JIT if enabled)
for _ in range(200):
    sum_squares(1000)

# Check if JIT is available in this build
print(f"Python version: {sys.version}")
print(f"JIT available: {hasattr(sys, '_jit')}")

# View specialized bytecode after warmup
print("\nSpecialized bytecode:")
dis.dis(sum_squares, adaptive=True)
```


## Resources

- [PEP 744 — JIT Compilation](https://peps.python.org/pep-0744/) — The PEP describing CPython's copy-and-patch JIT compiler design
- [What's New in Python 3.14](https://docs.python.org/3.14/whatsnew/3.14.html) — Official release notes for Python 3.14 including JIT and tail-call interpreter improvements

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*