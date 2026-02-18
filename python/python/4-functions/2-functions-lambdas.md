---
source_course: "python"
source_lesson: "python-functions-lambdas"
---

# First-Class Functions

In Python, functions are objects. They can be:
- Assigned to variables
- Passed as arguments to other functions
- Returned from functions
- Stored in data structures

```python
def greet(name):
    return f"Hello, {name}"

# Assign to variable
say_hello = greet
print(say_hello("Alice"))  # "Hello, Alice"

# Store in list
operations = [str.upper, str.lower, str.title]
for op in operations:
    print(op("hello world"))
```

## Higher-Order Functions

Functions that take or return functions:

```python
def apply_twice(func, value):
    return func(func(value))

def add_one(x):
    return x + 1

apply_twice(add_one, 5)  # 7
```

## Lambda Functions

Anonymous functions for simple operations:

```python
# Syntax: lambda arguments: expression
square = lambda x: x ** 2
add = lambda x, y: x + y

# Common use: sorting
users = [{"name": "Bob", "age": 30}, {"name": "Alice", "age": 25}]
sorted(users, key=lambda u: u["age"])
# [{"name": "Alice", ...}, {"name": "Bob", ...}]
```

## Closures

Functions that capture variables from their enclosing scope:

```python
def make_multiplier(n):
    def multiplier(x):
        return x * n  # n is captured from outer scope
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)

double(5)  # 10
triple(5)  # 15
```

## Code Examples

**Closure-based counter**

```python
# Practical closure example: counter factory
def make_counter():
    count = 0
    def counter():
        nonlocal count
        count += 1
        return count
    return counter

c1 = make_counter()
c2 = make_counter()
print(c1(), c1(), c1())  # 1, 2, 3
print(c2())              # 1 (independent counter)
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*