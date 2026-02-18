---
source_course: "python"
source_lesson: "python-generators-yield"
---

# Generators

Generators are functions that yield values one at a time, enabling lazy evaluation and memory efficiency.

## Basic Generator

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

## yield vs return

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

## yield from

Delegate to another generator:

```python
def chain(*iterables):
    for iterable in iterables:
        yield from iterable

list(chain([1, 2], [3, 4], [5]))  # [1, 2, 3, 4, 5]
```

## Generator Expressions

```python
# Like list comprehensions, but lazy
squares = (x ** 2 for x in range(1000000))
# No memory allocated for all values!

sum(x ** 2 for x in range(1000000))  # Efficient
```

## Sending Values to Generators

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*