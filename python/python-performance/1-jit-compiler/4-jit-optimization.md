---
source_course: "python-performance"
source_lesson: "python-performance-jit-optimization"
---

# Writing JIT-Friendly Code

## Introduction

Now that you understand how the adaptive interpreter specializes bytecodes and how the JIT compiles hot traces, it is time to learn how to write code that cooperates with these optimizers. Small changes in how you structure your code can make the difference between hitting fast specialized paths and falling back to slow generic execution.

## Key Concepts

- **Type Stability**: Writing functions that consistently receive the same types for each argument across all calls, enabling specialization to stick.
- **Hot Loop**: A loop that executes many iterations and becomes the primary target for JIT compilation.
- **Local Variable Optimization**: Referencing frequently used globals or attributes as local variables to reduce lookup overhead in hot loops.
- **Warmup**: The initial period of execution before specialization and JIT compilation activate, during which performance is slower.
- **Deoptimization**: When the JIT or adaptive interpreter encounters a type or condition it did not expect, forcing a fallback to slower generic execution.

## Real World Context

In a data processing pipeline that transforms millions of records, the inner loop is where your program spends 99% of its time. Writing that inner loop with stable types, local variable references, and simple attribute access patterns can yield 20-50% speed improvements on Python 3.14 without changing any algorithms.

## Deep Dive

### Type Stability

The JIT and adaptive interpreter work best when types are predictable:

```python
# Good: Type-stable — always receives list[int]
def sum_values(numbers: list[int]) -> int:
    total = 0
    for n in numbers:
        total += n  # Always int + int → BINARY_OP_ADD_INT
    return total

# Bad: Type-unstable — mixed types cause deoptimization
def sum_anything(items):
    total = 0
    for item in items:
        total += item  # int? float? str? Keeps de-specializing
    return total
```

The first function lets the interpreter specialize `BINARY_OP` to `BINARY_OP_ADD_INT` and keep it. The second function triggers repeated specialization and de-specialization, which is slower than never specializing at all.

### Optimizing Hot Loops

Focus your optimization effort on the innermost loops:

```python
import math

def compute_distances(points, origin):
    """Calculate distance from origin for each point."""
    ox, oy = origin  # Unpack once outside the loop
    sqrt = math.sqrt  # Local reference to avoid global lookup
    
    distances = []
    append = distances.append  # Avoid attribute lookup per iteration
    
    for px, py in points:
        dx = px - ox
        dy = py - oy
        append(sqrt(dx * dx + dy * dy))
    
    return distances
```

Each of these optimizations removes one dictionary lookup per iteration from the hot loop. Over millions of iterations, this adds up significantly.

### Local Variable Optimization

CPython accesses local variables with `LOAD_FAST` (a simple array index), while globals use `LOAD_GLOBAL` (a dictionary lookup):

```python
import math

# Slow: LOAD_GLOBAL on every iteration
def slow_sqrt_sum(values):
    total = 0.0
    for v in values:
        total += math.sqrt(v)  # Two lookups: 'math' + 'sqrt'
    return total

# Fast: LOAD_FAST on every iteration
def fast_sqrt_sum(values):
    sqrt = math.sqrt  # One-time LOAD_GLOBAL
    total = 0.0
    for v in values:
        total += sqrt(v)  # LOAD_FAST — simple array index
    return total
```

### Avoiding Dynamic Features in Hot Code

Dynamic features prevent specialization:

```python
# Avoid in hot loops:
getattr(obj, attr_name)        # Dynamic attribute access
obj.__dict__[key]              # Direct dict manipulation
exec(code_string)              # Dynamic code execution
globals()[name]                # Dynamic global lookup

# Prefer in hot loops:
obj.attribute                  # Direct attribute access
obj.method()                   # Direct method call
local_var                      # Local variable access
```

### Benchmarking with Warmup

Always include a warmup phase when benchmarking to allow specialization and JIT compilation:

```python
import time

def benchmark(func, *args, warmup=200, iterations=1000):
    """Benchmark with proper warmup for JIT/specialization."""
    # Warmup phase: triggers specialization and JIT
    for _ in range(warmup):
        func(*args)
    
    # Measurement phase
    start = time.perf_counter()
    for _ in range(iterations):
        func(*args)
    elapsed = time.perf_counter() - start
    
    avg_us = (elapsed / iterations) * 1_000_000
    print(f"{func.__name__}: {avg_us:.2f} us/call")
    return elapsed
```

## Common Pitfalls

- **Benchmarking without warmup**: Measuring a function on its first few calls captures unspecialized performance. Always run at least 100-200 warmup iterations before timing.
- **Over-applying local variable tricks**: Hoisting globals to locals only matters in tight inner loops with thousands of iterations. In code that runs once, the readability cost is not worth it.
- **Ignoring algorithmic complexity**: No amount of JIT-friendliness will save an O(n^2) algorithm from being slow on large inputs. Always fix the algorithm first, then micro-optimize hot paths.

## Best Practices

- **Profile first, then optimize hot paths**: Use `cProfile` or `timeit` to identify which functions and loops are actually hot before applying these techniques.
- **Separate type-specific logic into distinct functions**: Instead of one polymorphic function, write `process_ints()` and `process_floats()` so each can specialize independently.
- **Use `time.perf_counter()` for benchmarks**: It provides the highest-resolution timer available, which is essential for measuring microsecond-level differences.

## Summary

- Type stability is the most important factor: consistent types allow specialization to persist and the JIT to eliminate guards
- Hoist global lookups and attribute accesses to local variables in hot loops to replace `LOAD_GLOBAL` with `LOAD_FAST`
- Avoid dynamic features (`getattr`, `__dict__` access, `exec`) in performance-critical code paths
- Always include a warmup phase (200+ iterations) in benchmarks to account for specialization and JIT compilation time
- Focus optimization effort on inner loops where the program spends the most time; algorithmic improvements always come first

## Code Examples

**Comparing a function with global lookups in a hot loop versus one with local variable optimization, including proper warmup before benchmarking**

```python
import time
import math

def slow_distance(points):
    """Unoptimized: global lookups on every iteration."""
    total = 0.0
    for x, y in points:
        total += math.sqrt(x * x + y * y)
    return total

def fast_distance(points):
    """Optimized: local variable for math.sqrt."""
    sqrt = math.sqrt
    total = 0.0
    for x, y in points:
        total += sqrt(x * x + y * y)
    return total

# Generate test data
test_points = [(float(i), float(i + 1)) for i in range(100_000)]

# Warmup both functions
for _ in range(50):
    slow_distance(test_points)
    fast_distance(test_points)

# Benchmark
for func in [slow_distance, fast_distance]:
    start = time.perf_counter()
    for _ in range(10):
        func(test_points)
    elapsed = time.perf_counter() - start
    print(f"{func.__name__}: {elapsed:.4f}s")
```


## Resources

- [Python Performance Tips](https://docs.python.org/3.14/howto/perf_profiling.html) — Official guide to profiling and performance tuning in Python

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*