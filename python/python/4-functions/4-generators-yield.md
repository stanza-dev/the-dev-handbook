---
source_course: "python"
source_lesson: "python-generators-yield"
---

# Generators & Yield

## Introduction
Generators let you produce sequences of values lazily, one at a time, without holding the entire sequence in memory. This lesson covers `yield`, `yield from`, generator expressions, and the advanced `.send()` protocol.

## Key Concepts
- **Generator function**: A function containing `yield` that returns a generator iterator when called.
- **`yield`**: Pauses the function, produces a value, and preserves all local state for resumption.
- **`yield from`**: Delegates to a sub-generator, forwarding values in both directions.
- **Generator expression**: A parenthesized comprehension that produces values lazily.
- **`.send(value)`**: Sends a value into a paused generator, resuming it and assigning the sent value to the `yield` expression.

## Real World Context
Generators power Python's async framework under the hood and are essential for processing large datasets that do not fit in memory -- reading log files line by line, streaming API responses, or feeding batches to a machine learning model. Any time you see `for line in open(file)`, you are using a generator.

## Deep Dive

### Basic Generator

```python
def count_up_to(n):
    count = 1
    while count <= n:
        yield count
        count += 1

# Using the generator
for num in count_up_to(5):
    print(num)  # 1, 2, 3, 4, 5

# Or manually
gen = count_up_to(3)
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
print(next(gen))  # StopIteration!
```

### yield vs return

- `return` terminates the function and returns a value
- `yield` pauses the function, preserving state, and produces a value

```python
def fibonacci():
    a, b = 0, 1
    while True:  # Infinite generator!
        yield a
        a, b = b, a + b

fib = fibonacci()
[next(fib) for _ in range(10)]  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### yield from

Delegate to another generator:

```python
def chain(*iterables):
    for iterable in iterables:
        yield from iterable

list(chain([1, 2], [3, 4], [5]))  # [1, 2, 3, 4, 5]
```

### Generator Expressions

```python
# Like list comprehensions, but lazy
squares = (x ** 2 for x in range(1000000))
# No memory allocated for all values!

sum(x ** 2 for x in range(1000000))  # Efficient
```

### Sending Values to Generators

```python
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value

acc = accumulator()
next(acc)         # Initialize: 0
acc.send(10)      # 10
acc.send(5)       # 15
```

## Common Pitfalls
1. **Trying to reuse an exhausted generator** -- Once a generator has raised `StopIteration`, calling `next()` again keeps raising it. You must create a new generator by calling the function again.
2. **Forgetting to call `next()` before `.send()`** -- A generator must be primed (advanced to the first `yield`) before you can send values into it. The first call must be `next(gen)` or `gen.send(None)`.
3. **Converting a generator to a list unnecessarily** -- `list(gen)` defeats the purpose of lazy evaluation by materializing all values at once. Only convert when you truly need random access.

## Best Practices
1. **Use generators for pipeline-style data processing** -- Chain generators to filter, transform, and aggregate data without intermediate lists.
2. **Prefer `yield from` over a manual loop** -- `yield from iterable` is cleaner and more efficient than `for item in iterable: yield item`.

## Summary
- Generator functions use `yield` to produce values lazily, one at a time.
- `yield` preserves all local state; `return` terminates the generator.
- `yield from` delegates to sub-generators cleanly.
- Generator expressions provide lazy one-liner alternatives to list comprehensions.
- Always prime a generator with `next()` before using `.send()`.

## Code Examples

**Memory-efficient file reading**

```python
# Practical: reading large files line by line
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()

# Process without loading entire file into memory
for line in read_large_file("huge_file.txt"):
    if "error" in line.lower():
        print(line)
```


## Resources

- [Generators](https://docs.python.org/3.14/howto/functional.html#generators) — Official Python 3.14 guide on generators, yield expressions, and generator-based patterns

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*