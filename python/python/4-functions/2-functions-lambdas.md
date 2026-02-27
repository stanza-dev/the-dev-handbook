---
source_course: "python"
source_lesson: "python-functions-lambdas"
---

# First-Class Functions & Lambdas

## Introduction
In Python, functions are full-fledged objects -- you can assign them to variables, store them in lists, and pass them to other functions. This lesson covers first-class functions, lambda expressions, higher-order functions, and closures.

## Key Concepts
- **First-class functions**: Functions are objects that can be assigned, passed, and returned like any other value.
- **Higher-order function**: A function that takes another function as an argument or returns one.
- **Lambda**: An anonymous, single-expression function created with the `lambda` keyword.
- **Closure**: A function that captures and remembers variables from its enclosing scope.

## Real World Context
Callbacks, event handlers, and middleware chains all depend on first-class functions. Frameworks like Flask use decorators (built on closures) to register routes, and libraries like `sorted()` accept `key` functions to customize ordering. Understanding closures is also essential for writing factories, memoizers, and configuration builders.

## Deep Dive

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

### Higher-Order Functions

Functions that take or return functions:

```python
def apply_twice(func, value):
    return func(func(value))

def add_one(x):
    return x + 1

apply_twice(add_one, 5)  # 7
```

### Lambda Functions

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

### Closures

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

## Common Pitfalls
1. **Writing multi-line logic in a lambda** -- Lambdas are limited to a single expression. If you need statements, conditions across multiple lines, or error handling, use a regular `def` function.
2. **Late binding in closures over loop variables** -- A closure created inside a loop captures the variable itself, not its current value. All closures end up sharing the final loop value. Fix by using a default argument: `lambda x, n=n: x * n`.
3. **Assigning lambdas to names** -- `square = lambda x: x**2` is less readable than `def square(x): return x**2` and loses the function name in tracebacks. PEP 8 discourages this pattern.

## Best Practices
1. **Use lambdas only for short, throwaway functions** -- A `key=lambda x: x.name` in a `sorted()` call is ideal. Anything longer deserves a named function.
2. **Prefer `operator` module functions over trivial lambdas** -- `operator.itemgetter('age')` is faster and clearer than `lambda u: u['age']` for simple attribute or item access.

## Summary
- Functions in Python are first-class objects: assignable, passable, and returnable.
- Higher-order functions accept or return other functions, enabling powerful abstractions.
- Lambdas are single-expression anonymous functions, best used as short callbacks.
- Closures capture variables from enclosing scopes -- beware of late binding in loops.
- Prefer named functions over lambdas for anything beyond a trivial one-liner.

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


## Resources

- [Lambda Expressions](https://docs.python.org/3.14/tutorial/controlflow.html#lambda-expressions) — Official Python 3.14 tutorial on lambda functions and first-class function concepts

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*