---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-inspect-module"
---

# The Inspect Module

## Introduction

The `inspect` module is your window into the internals of running Python code. It lets you examine live objects at runtime -- retrieving function signatures, querying object types, walking the call stack, and even reading source code. Whether you are building a plugin system, writing a debugger, or creating a framework, `inspect` is the foundation of Python introspection.

## Key Concepts

- **`inspect.signature()`** -- Returns a `Signature` object describing a function's parameters, defaults, and return annotation.
- **Type-checking predicates** -- Functions like `isfunction()`, `isclass()`, `ismodule()`, and `isbuiltin()` that identify what kind of object you are looking at.
- **`inspect.ispackage()`** -- New in Python 3.14, checks whether a module is a package (has a `__path__` attribute).
- **Stack frame introspection** -- Functions like `currentframe()` let you walk the call stack and inspect caller information.
- **Source retrieval** -- `getsource()` and `getfile()` let you read the actual source code of live objects.

## Real World Context

Imagine you are building a web framework that automatically registers route handlers. You need to inspect each handler function to determine its parameter names, types, and defaults so you can wire up request data correctly. The `inspect` module makes this possible without requiring developers to write boilerplate registration code -- frameworks like FastAPI rely heavily on `inspect.signature()` to achieve this exact pattern.

## Deep Dive

The most common use of the `inspect` module is retrieving function signatures. The `signature()` function returns a `Signature` object that gives you structured access to every parameter, its default value, and its type annotation.

```python
import inspect

def my_func(a: int, b: int = 10) -> int:
    """Add two numbers."""
    return a + b

sig = inspect.signature(my_func)
print(sig)  # (a: int, b: int = 10) -> int
print(sig.parameters['b'].default)  # 10
print(sig.parameters['b'].annotation)  # <class 'int'>
print(sig.return_annotation)  # <class 'int'>
```

The `Signature` object provides a dictionary-like `.parameters` attribute where each key is the parameter name and each value is a `Parameter` object with `.default`, `.annotation`, and `.kind` attributes.

Python 3.14 adds a new `annotation_format` parameter to `inspect.signature()`. This parameter accepts values from the `annotationlib.Format` enum and controls whether annotations are returned as evaluated objects or as raw strings.

```python
import inspect
from annotationlib import Format

def greet(name: str) -> str:
    return f"Hello, {name}"

# Default: returns evaluated annotation objects
sig = inspect.signature(greet, annotation_format=Format.VALUE)
print(sig.parameters['name'].annotation)  # <class 'str'>

# Returns string representations instead
sig = inspect.signature(greet, annotation_format=Format.STRING)
print(sig.parameters['name'].annotation)  # 'str'
```

This is particularly useful when you want to serialize or display signatures without triggering evaluation of forward references.

Beyond signatures, `inspect` provides a family of predicate functions for querying what type of object you are dealing with. These are essential when writing code that handles heterogeneous collections of objects.

```python
import inspect
import os

print(inspect.isfunction(my_func))  # True
print(inspect.isclass(int))         # True
print(inspect.ismodule(os))         # True
print(inspect.isbuiltin(print))     # True
```

Each predicate returns a simple boolean, making them ideal for filtering and dispatching logic.

Python 3.14 introduces `inspect.ispackage()`, which checks whether a module is a package -- that is, whether it has a `__path__` attribute. This is a subtle but important distinction: a package is a directory-based module that can contain sub-modules, while a regular module is a single `.py` or `.so` file.

```python
import inspect
import os
import json

import email
print(inspect.ispackage(email))  # True  (email is a package)
print(inspect.ispackage(json))   # True  (json is a package)

import math
print(inspect.ispackage(math))   # False (math is a single module)
print(inspect.ispackage(os))     # False (os is a module, not a package)
```

Notice that `os` returns `False` even though it feels like a large standard library module -- it is implemented as a single module file, not a package directory.

Stack frame introspection is one of the more powerful (and delicate) features of `inspect`. You can access the current execution frame and walk backward through the call stack to discover who called the current function, from which line, and in which file.

```python
def who_called_me():
    frame = inspect.currentframe()
    caller = frame.f_back
    print(f"Called by {caller.f_code.co_name}")
    print(f"At line {caller.f_lineno}")
    print(f"In file {caller.f_code.co_filename}")

def main():
    who_called_me()

main()
# Called by main
# At line 10
# In file example.py
```

The `f_back` attribute points to the caller's frame, and each frame carries a `f_code` object with metadata about the function, plus `f_lineno` for the current line number.

Finally, `inspect` can retrieve the actual source code of any object defined in a `.py` file. This is invaluable for debugging tools, documentation generators, and interactive notebooks.

```python
import inspect

def example():
    return 42

print(inspect.getsource(example))
# def example():
#     return 42

print(inspect.getfile(example))  # Path to the file
```

Note that `getsource()` only works for objects defined in Python source files -- it will raise an `OSError` for built-in functions or objects defined in C extensions.

## Common Pitfalls

- **Holding references to stack frames** -- Storing frame objects (from `inspect.currentframe()`) can create reference cycles that prevent garbage collection. Always delete frame references when you are done with them, or use `try/finally` blocks.
- **Calling `getsource()` on built-ins** -- Functions like `print` or `len` are implemented in C, not Python. Calling `inspect.getsource(print)` raises `OSError`. Always check with `inspect.isbuiltin()` first.
- **Confusing `ispackage()` with `ismodule()`** -- Every package is a module, but not every module is a package. `ismodule(email)` returns `True` AND `ispackage(email)` returns `True`, but `ismodule(math)` returns `True` while `ispackage(math)` returns `False`.

## Best Practices

- **Use `inspect.signature()` instead of the deprecated `getfullargspec()`** -- The `Signature` API is more powerful, handles all parameter kinds correctly, and in Python 3.14 supports the `annotation_format` parameter.
- **Prefer `inspect` predicates over manual `isinstance` checks** -- Using `inspect.isclass(obj)` is clearer and more robust than `isinstance(obj, type)`, and communicates intent better.
- **Clean up frame references immediately** -- If you use `currentframe()` or `stack()`, wrap the usage in a `try/finally` block and `del` the frame variable to avoid reference cycles.

## Summary

- `inspect.signature()` gives you structured access to any function's parameters, defaults, and annotations -- and in Python 3.14, supports `annotation_format` for controlling how annotations are returned.
- Type-checking predicates (`isfunction`, `isclass`, `ismodule`, `isbuiltin`) let you query what kind of object you are working with.
- Python 3.14 adds `inspect.ispackage()` to distinguish packages (like `email`) from plain modules (like `math` and `os`).
- Stack frame introspection via `currentframe()` and `f_back` lets you walk the call stack, but you must clean up frame references to avoid memory leaks.
- `getsource()` retrieves the source code of Python-defined objects, but does not work on C-level built-ins.

## Code Examples

**Inspecting Stack Frames**

```python
import inspect

def who_called_me():
    frame = inspect.currentframe()
    caller = frame.f_back
    print(f"Called by {caller.f_code.co_name}")

def main():
    who_called_me()

main()  # Called by main
```

**Detailed signature inspection**

```python
import inspect

def process(a: int, b: str = 'hello', *args, **kwargs) -> bool:
    pass

sig = inspect.signature(process)
for name, param in sig.parameters.items():
    print(f"{name}: kind={param.kind.name}, "
          f"default={param.default}, "
          f"annotation={param.annotation}")
# a: kind=POSITIONAL_OR_KEYWORD, default=<class 'inspect._empty'>, annotation=<class 'int'>
# b: kind=POSITIONAL_OR_KEYWORD, default=hello, annotation=<class 'str'>
# args: kind=VAR_POSITIONAL, ...
# kwargs: kind=VAR_KEYWORD, ...
```


## Resources

- [inspect — Inspect live objects (Python 3.14)](https://docs.python.org/3.14/library/inspect.html) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*