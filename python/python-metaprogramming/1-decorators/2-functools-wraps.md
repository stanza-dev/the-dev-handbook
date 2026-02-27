---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-functools-wraps"
---

# Preserving Function Metadata

## Introduction

When you decorate a function, the wrapper replaces the original in every way -- including its name, docstring, and type annotations. This causes real problems for debugging, documentation generation, and introspection tools. The `functools.wraps` decorator solves this by copying the original function's metadata onto the wrapper, and Python 3.14 brings important changes to how annotations are handled.

## Key Concepts

- **`functools.wraps(func)`** -- A decorator applied to the wrapper function that copies key metadata attributes from the original `func` onto the wrapper.
- **`__wrapped__`** -- A special attribute set by `functools.wraps` that stores a reference to the original unwrapped function.
- **`__annotate__`** -- The annotation evaluator function introduced by PEP 649 in Python 3.14, which lazily produces `__annotations__` on demand.
- **`__type_params__`** -- Type parameter metadata for generic functions, available since Python 3.12 (PEP 695).

## Real World Context

In production code, broken metadata is more than a cosmetic issue. Documentation generators like Sphinx rely on `__name__` and `__doc__` to produce API docs. Debugging tools and profilers display `__name__` and `__qualname__` in stack traces. Serialization frameworks like Pickle use `__qualname__` to locate functions. Type checkers and IDE tooling inspect annotations. Without `functools.wraps`, all of these tools see the wrapper's metadata instead of the original function's, leading to confusing docs, misleading stack traces, and broken serialization.

## Deep Dive

To understand the problem, consider a simple decorator that does not use `functools.wraps`:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone."""
    return f"Hello, {name}"

print(greet.__name__)  # "wrapper" - Not "greet"!
print(greet.__doc__)   # None - Docstring lost!
```

After decoration, `greet` is actually `wrapper`. Its `__name__` is `"wrapper"`, its docstring is `None`, and any annotations are gone. This is because the decorator returned a brand-new function object that has no knowledge of the original.

The fix is to apply `@functools.wraps(func)` to the wrapper, which copies all the relevant metadata:

```python
import functools

def my_decorator(func):
    @functools.wraps(func)  # Copy metadata from func to wrapper
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone."""
    return f"Hello, {name}"

print(greet.__name__)  # "greet" - Correct!
print(greet.__doc__)   # "Greet someone." - Preserved!
```

Now `greet.__name__` is `"greet"`, the docstring is preserved, and `greet.__wrapped__` gives access to the original unwrapped function.

`functools.wraps` copies the following attributes from the original function to the wrapper:

- `__module__`: Module name
- `__name__`: Function name
- `__qualname__`: Qualified name
- `__annotations__`: Type annotations dictionary
- `__type_params__`: Type parameters (since Python 3.12, PEP 695)
- `__doc__`: Docstring

It also updates `__dict__` (merging function attributes) and sets `__wrapped__` to the original function. In Python 3.14, `__annotate__` (PEP 649) is also copied internally if present.

A notable change in Python 3.14 is PEP 649 (deferred evaluation of annotations). Python 3.14 adds `__annotate__` -- a callable that lazily evaluates annotations on demand. `functools.wraps` copies `__annotate__` alongside `__annotations__` when present. This means annotations are no longer eagerly evaluated at function definition time, improving startup performance and eliminating forward-reference issues. Accessing `__annotations__` on wrapped functions still works because Python 3.14 generates it on the fly from `__annotate__`.

Similarly, since Python 3.12, `functools.wraps` also copies `__type_params__`, ensuring that generic function type parameters (defined with the `type` statement or `TypeVar`) are preserved through decoration.

## Common Pitfalls

- **Forgetting `functools.wraps` entirely.** This is the most common mistake. Every decorator you write should use `@functools.wraps(func)` on its wrapper unless you have a specific reason not to.
- **Assuming `__wrapped__` is always the true original.** If multiple decorators are stacked, each one sets `__wrapped__` to its immediate input -- not the bottom-most original. To unwrap fully, use `inspect.unwrap(func)`, which follows the `__wrapped__` chain.
- **Mutating `__annotations__` directly on Python 3.14+.** While reading `__annotations__` still works (it is generated from `__annotate__` on demand), directly mutating `__annotations__` may not behave as expected with deferred evaluation. Use `annotationlib.get_annotations()` for reliable access.

## Best Practices

- Always apply `@functools.wraps(func)` to your wrapper function. It costs nothing and prevents a wide class of metadata-related bugs.
- Use `inspect.unwrap(func)` when you need to access the true original function through multiple layers of decoration.
- When writing class-based decorators, use `functools.update_wrapper(self, func)` in `__init__` to achieve the same metadata copying.

## Summary

- Without `functools.wraps`, decorated functions lose their name, docstring, and annotations.
- `@functools.wraps(func)` copies all key metadata from the original function to the wrapper.
- In Python 3.14, `functools.wraps` also copies `__annotate__` (PEP 649) alongside `__annotations__`, supporting deferred annotation evaluation.
- Since Python 3.12, `__type_params__` is also preserved for generic functions (PEP 695).
- The `__wrapped__` attribute provides access to the original function for introspection and unwrapping.

## Code Examples

**Complete decorator with wraps**

```python
import functools

def debug(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}({args}, {kwargs})")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@debug
def add(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

# Metadata preserved
print(add.__name__)        # add
print(add.__doc__)         # Add two numbers.
print(add.__wrapped__)     # <function add at 0x...> (original function)
```


## Resources

- [functools.wraps](https://docs.python.org/3.14/library/functools.html#functools.wraps) — Official docs for functools.wraps
- [PEP 649 — Deferred Evaluation of Annotations](https://peps.python.org/pep-0649/) — PEP that introduced deferred evaluation of annotations via __annotate__ in Python 3.14

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*