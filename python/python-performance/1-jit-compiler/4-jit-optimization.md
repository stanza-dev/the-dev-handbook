---
source_course: "python-performance"
source_lesson: "python-performance-jit-optimization"
---

# Optimizing for the JIT

## Type Stability

The JIT works best with predictable types:

```python
# Good: Stable types
def process(items: list[int]) -> int:
    return sum(items)

# Bad: Mixed types cause deoptimization
def process_mixed(items):
    result = 0  # int
    for item in items:
        result = result + item  # item could be anything
    return result
```

## Hot Loops

Focus optimization on frequently executed code:

```python
# The inner loop is "hot" - JIT focuses here
for _ in range(1000):
    for i in range(10000):  # Hot!
        total += i
```

## Avoid Dynamic Features in Hot Code

```python
# Slow: Dynamic attribute access
for _ in range(1000000):
    x = obj.__dict__['value']  # Hard to optimize

# Fast: Direct attribute
for _ in range(1000000):
    x = obj.value  # JIT can specialize
```

## Local Variables are Faster

```python
# Slow: Global lookup each iteration
import math

def slow():
    for i in range(1000000):
        math.sqrt(i)  # Global lookup

# Fast: Local reference
def fast():
    sqrt = math.sqrt  # One lookup
    for i in range(1000000):
        sqrt(i)
```

## Measuring JIT Impact

```python
import time

def benchmark(func, *args, warmup=100, iterations=1000):
    # Warm up (trigger JIT)
    for _ in range(warmup):
        func(*args)
    
    # Measure
    start = time.perf_counter()
    for _ in range(iterations):
        func(*args)
    elapsed = time.perf_counter() - start
    
    return elapsed / iterations
```

## Code Examples

**Benchmarking**

```python
import time

def sum_naive(n):
    total = 0
    for i in range(n):
        total += i
    return total

def sum_builtin(n):
    return sum(range(n))

# Benchmark
n = 1_000_000

start = time.perf_counter()
for _ in range(10):
    sum_naive(n)
print(f"Naive: {time.perf_counter() - start:.3f}s")

start = time.perf_counter()
for _ in range(10):
    sum_builtin(n)
print(f"Built-in: {time.perf_counter() - start:.3f}s")
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*