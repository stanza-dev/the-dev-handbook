---
source_course: "python"
source_lesson: "python-function-basics"
---

# Function Fundamentals

## Introduction
Functions are the primary way to organize and reuse code in Python. This lesson covers everything from basic definitions and parameter types to type hints and Python 3.14's deferred annotations that eliminate the need for quoted forward references.

## Key Concepts
- **`def` statement**: Defines a named, reusable block of code.
- **Positional and keyword arguments**: Two ways to pass values to functions.
- **`*args` / `**kwargs`**: Collect variable numbers of positional or keyword arguments.
- **Positional-only (`/`) and keyword-only (`*`) parameters**: Enforce how callers pass arguments.
- **Type hints**: Optional annotations that document expected types and enable static analysis.
- **Deferred annotations (PEP 649)**: Python 3.14 feature that removes the need for quoted forward references.

## Real World Context
Well-designed function signatures are the contract between your code and its callers. In large codebases, keyword-only parameters prevent accidental argument ordering bugs, and type hints catch misuse before runtime. Libraries like FastAPI and Pydantic rely heavily on function signatures and type hints to auto-generate documentation and validate input.

## Deep Dive

### Basic Syntax

```python
def function_name(parameters):
    """Docstring describing the function."""
    # function body
    return value
```

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

### Type Hints

```python
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str, times: int = 1) -> list[str]:
    return [f"Hello, {name}!"] * times
```

### Deferred Annotations (Python 3.14+)

In Python 3.14, annotations are no longer eagerly evaluated (PEP 649). This means forward references work without quotes:

```python
# Before 3.14 -- needed quotes for forward references
def create() -> "MyClass":
    return MyClass()

# Python 3.14+ -- no quotes needed
def create() -> MyClass:
    return MyClass()

class MyClass:
    pass
```

## Common Pitfalls
1. **Using a mutable default argument** -- `def add_item(item, lst=[]):` shares the same list across all calls. Use `None` as the default and create a new list inside the function: `if lst is None: lst = []`.
2. **Confusing positional-only and keyword-only separators** -- `/` ends positional-only parameters; `*` begins keyword-only parameters. Mixing them up causes confusing TypeErrors.
3. **Ignoring type hints in team projects** -- Type hints are not enforced at runtime, but skipping them means you lose the benefits of static analysis tools like mypy and IDE autocomplete.

## Best Practices
1. **Use keyword-only arguments for boolean flags** -- `def run(*, verbose=False)` prevents callers from accidentally passing `True` as a positional argument to the wrong parameter.
2. **Always write a docstring for public functions** -- A one-line docstring is enough for simple functions; use Google or NumPy style for complex ones.

## Summary
- Functions are defined with `def` and can accept positional, keyword, `*args`, and `**kwargs` parameters.
- Use `/` for positional-only and `*` for keyword-only to control how callers pass arguments.
- Type hints document expected types and enable static analysis without runtime overhead.
- Python 3.14's deferred annotations (PEP 649) eliminate the need for quoted forward references.
- Never use mutable default arguments; use `None` and create fresh objects inside the function.

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


## Resources

- [Defining Functions](https://docs.python.org/3.14/tutorial/controlflow.html#defining-functions) — Official Python 3.14 tutorial on function definitions, parameters, and type hints

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*