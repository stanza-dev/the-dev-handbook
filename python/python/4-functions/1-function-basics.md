---
source_course: "python"
source_lesson: "python-function-basics"
---

# Defining Functions

## Basic Syntax

```python
def function_name(parameters):
    """Docstring describing the function."""
    # function body
    return value
```

## Parameters and Arguments

### Positional and Keyword Arguments

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice")                    # "Hello, Alice!"
greet("Bob", "Hi")                # "Hi, Bob!"
greet(greeting="Hey", name="Eve") # "Hey, Eve!"
```

### *args and **kwargs

```python
def flexible(*args, **kwargs):
    print(f"Positional: {args}")
    print(f"Keyword: {kwargs}")

flexible(1, 2, 3, name="Alice", age=30)
# Positional: (1, 2, 3)
# Keyword: {'name': 'Alice', 'age': 30}
```

### Keyword-Only Arguments (after *)

```python
def configure(*, debug=False, verbose=False):
    # debug and verbose must be passed as keywords
    pass

configure(debug=True)  # OK
configure(True)        # TypeError!
```

### Positional-Only Arguments (before /)

```python
def pow(x, y, /):
    # x and y must be positional
    return x ** y

pow(2, 10)    # OK: 1024
pow(x=2, y=10)  # TypeError!
```

## Type Hints

```python
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str, times: int = 1) -> list[str]:
    return [f"Hello, {name}!"] * times
```

## Code Examples

**Advanced function signatures**

```python
# Combining parameter types
def api_call(
    endpoint,           # Positional or keyword
    /,                  # Everything before is positional-only
    *args,              # Extra positional args
    timeout=30,         # Keyword argument with default
    **kwargs            # Extra keyword args
):
    pass

# Modern type hints
def process[T](items: list[T]) -> list[T]:
    return [item for item in items if item]
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*