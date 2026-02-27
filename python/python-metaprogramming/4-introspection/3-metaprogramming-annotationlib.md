---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-annotationlib"
---

# The annotationlib Module (Python 3.14)

## Introduction

Python 3.14 introduces the `annotationlib` module, a fundamental shift in how type annotations work under the hood. Instead of evaluating annotations eagerly when a class or function is defined, Python now defers evaluation until the annotations are actually accessed. This eliminates the long-standing forward reference problem and gives developers fine-grained control over how annotations are retrieved and processed.

## Key Concepts

- **Deferred evaluation** -- Annotations are stored as lightweight functions and evaluated on demand, not at definition time.
- **`annotationlib.Format`** -- An enum with three modes (`VALUE`, `FORWARDREF`, `STRING`) that controls how annotations are returned.
- **`annotationlib.get_annotations()`** -- The new recommended way to retrieve annotations from any class or function.
- **`ForwardRef`** -- A placeholder object for annotations that cannot be resolved yet, with a `__forward_arg__` attribute containing the original name.
- **PEP 649 & PEP 749** -- The proposals that introduced deferred evaluation and the `annotationlib` module, respectively.

## Real World Context

If you have ever used `dataclasses`, `pydantic`, or an ORM like SQLAlchemy, you have hit the forward reference problem: a class that references itself (like a tree node pointing to child nodes of the same type) requires ugly string quotes around the type name. With `annotationlib`, frameworks can now access annotations safely regardless of definition order. Libraries like `pydantic` can use `Format.FORWARDREF` to get placeholder objects for unresolved types instead of crashing with `NameError`.

## Deep Dive

Before Python 3.14, forward references were a constant source of friction. If a class referenced itself in its annotations, Python would try to evaluate the annotation immediately and fail because the class was not fully defined yet.

```python
# Before Python 3.14: forward reference fails without quotes
class Tree:
    left: Tree   # NameError! Tree isn't defined yet
    right: Tree

# Before: workaround with strings
class Tree:
    left: 'Tree'   # Works, but now it's a string, not a type
    right: 'Tree'

# Python 3.14: just works!
class Tree:
    left: Tree   # Deferred — not evaluated until accessed
    right: Tree
```

The old workaround -- `from __future__ import annotations` (PEP 563) -- turned all annotations into strings, which caused problems for tools that needed actual type objects. PEP 649 solves this by storing annotations as a lightweight function that is evaluated on demand, giving you real type objects when you ask for them.

The `annotationlib.Format` enum provides three modes for retrieving annotations. Each mode serves a different use case, and choosing the right one depends on how much resolution you need.

```python
import annotationlib
from annotationlib import Format

class User:
    name: str
    friends: list[User]  # Forward ref — works in 3.14!

# FORMAT.VALUE — fully evaluated Python objects (default)
anns = annotationlib.get_annotations(User, format=Format.VALUE)
print(anns)  # {'name': <class 'str'>, 'friends': list[User]}

# FORMAT.FORWARDREF — unresolvable names become ForwardRef objects
anns = annotationlib.get_annotations(User, format=Format.FORWARDREF)
print(anns)  # {'name': <class 'str'>, 'friends': list[User]}

# FORMAT.STRING — everything as source-code strings
anns = annotationlib.get_annotations(User, format=Format.STRING)
print(anns)  # {'name': 'str', 'friends': 'list[User]'}
```

`Format.VALUE` gives you fully resolved type objects and is the default. `Format.FORWARDREF` is the safe middle ground -- it returns real types where possible and `ForwardRef` placeholders where resolution would fail. `Format.STRING` returns raw source strings and never raises.

The `annotationlib.get_annotations()` function is the new recommended way to retrieve annotations. It replaces `typing.get_type_hints()` for most use cases, with a cleaner API and explicit format control.

```python
import annotationlib

def process(data: list[int], flag: bool = True) -> dict[str, int]:
    pass

# Get annotations with full control
anns = annotationlib.get_annotations(process, format=Format.VALUE)
print(anns)
# {'data': list[int], 'flag': <class 'bool'>, 'return': dict[str, int]}
```

This works identically for functions and classes, with consistent behavior across both.

To appreciate the improvement, compare the old and new approaches for accessing annotations on a class with an unresolved forward reference.

Before Python 3.14, `typing.get_type_hints()` would crash if a referenced type was not yet defined.

```python
import typing

class Model:
    field: 'SomeType'  # String annotation

# typing.get_type_hints() tries to resolve strings
# Raises NameError if SomeType isn't defined
hints = typing.get_type_hints(Model)
```

With Python 3.14, you have multiple safe options depending on your needs.

```python
import annotationlib
from annotationlib import Format

class Model:
    field: SomeType  # No quotes needed!

# Safe access — ForwardRef if not yet resolvable
hints = annotationlib.get_annotations(Model, format=Format.FORWARDREF)
print(hints)  # {'field': ForwardRef('SomeType')}

# String access — always safe, never raises
hints = annotationlib.get_annotations(Model, format=Format.STRING)
print(hints)  # {'field': 'SomeType'}
```

The `Format.FORWARDREF` approach is especially powerful because it gives you a `ForwardRef` object you can resolve later, rather than failing outright.

The `annotationlib` module also integrates directly with `inspect.signature()` through the `annotation_format` parameter. This means you can control annotation resolution when inspecting function signatures too.

```python
import inspect
from annotationlib import Format

def greet(name: str) -> str:
    return f"Hello, {name}"

sig = inspect.signature(greet, annotation_format=Format.STRING)
print(sig)  # (name: 'str') -> 'str'
```

This returns a signature where all annotations are strings, which is useful for serialization or display purposes.

When you use `Format.FORWARDREF`, any annotation that cannot be resolved becomes a `ForwardRef` object. You can inspect its `__forward_arg__` attribute to see the original name, and call `evaluate()` to resolve it once the type becomes available.

```python
from annotationlib import ForwardRef, Format
import annotationlib

class Node:
    child: Node  # Deferred

anns = annotationlib.get_annotations(Node, format=Format.FORWARDREF)
ref = anns.get('child')
if isinstance(ref, ForwardRef):
    print(ref.__forward_arg__)  # 'Node'
    # Evaluate when the type is available:
    resolved = ref.evaluate(globals=globals())
    print(resolved)  # <class 'Node'>
```

The `__forward_arg__` attribute contains the string name of the unresolved type, and `evaluate()` takes a `globals` dictionary to look up the actual class. This two-step approach gives you full control over when and how forward references are resolved.

## Common Pitfalls

- **Using `Format.VALUE` with truly undefined types** -- If the referenced type genuinely does not exist yet (not just a forward reference to the current class), `Format.VALUE` will raise `NameError`. Use `Format.FORWARDREF` when you are not sure all types are available.
- **Assuming `__annotations__` behaves the same as before** -- In Python 3.14, accessing `__annotations__` directly triggers deferred evaluation. Code that catches `NameError` from `__annotations__` access may need updating since the behavior has changed.
- **Mixing `typing.get_type_hints()` with `annotationlib`** -- While `typing.get_type_hints()` still works, it does not support the `Format` enum. For new Python 3.14 code, prefer `annotationlib.get_annotations()` for consistency.

## Best Practices

- **Use `annotationlib.get_annotations()` instead of `typing.get_type_hints()`** -- The new function provides explicit format control, consistent behavior, and better integration with deferred evaluation.
- **Default to `Format.FORWARDREF` in library code** -- If you are writing a library that inspects user annotations, `FORWARDREF` is the safest choice because it never raises on unresolved types while still giving you real type objects where possible.
- **Use `Format.STRING` for display and serialization** -- When you need annotations as strings (for documentation, logging, or API schemas), `Format.STRING` is guaranteed to succeed and gives clean source-code representations.

## Summary

- Python 3.14 defers annotation evaluation until access time, eliminating the forward reference problem without `from __future__ import annotations`.
- The `annotationlib.Format` enum provides three retrieval modes: `VALUE` (fully evaluated), `FORWARDREF` (safe placeholders), and `STRING` (raw source strings).
- `annotationlib.get_annotations()` is the new recommended replacement for `typing.get_type_hints()`, with explicit format control.
- `ForwardRef` objects expose `__forward_arg__` for the type name and `evaluate()` for deferred resolution.
- The `inspect.signature()` function integrates with `annotationlib` via the `annotation_format` parameter.

## Code Examples

**Deferred annotations with annotationlib.Format**

```python
import annotationlib
from annotationlib import Format

class LinkedList:
    value: int
    next: LinkedList  # Forward ref — just works in 3.14!

# Three ways to read annotations
for fmt in (Format.VALUE, Format.FORWARDREF, Format.STRING):
    anns = annotationlib.get_annotations(LinkedList, format=fmt)
    print(f"{fmt.name}: {anns}")

# VALUE:      {'value': <class 'int'>, 'next': <class 'LinkedList'>}
# FORWARDREF: {'value': <class 'int'>, 'next': <class 'LinkedList'>}
# STRING:     {'value': 'int', 'next': 'LinkedList'}
```

**Combining annotationlib with inspect.signature()**

```python
import annotationlib
from annotationlib import Format
import inspect

def transform(data: list[int], *, strict: bool = False) -> dict[str, int]:
    """Transform data."""
    pass

# Using annotationlib directly
anns = annotationlib.get_annotations(transform, format=Format.STRING)
print(anns)
# {'data': 'list[int]', 'strict': 'bool', 'return': 'dict[str, int]'}

# Using inspect.signature with annotation_format
sig = inspect.signature(transform, annotation_format=Format.STRING)
for name, param in sig.parameters.items():
    print(f"{name}: {param.annotation}")
# data: 'list[int]'
# strict: 'bool'
```


## Resources

- [annotationlib — Utilities for annotations (Python 3.14)](https://docs.python.org/3.14/library/annotationlib.html) — undefined
- [inspect — Inspect live objects (Python 3.14)](https://docs.python.org/3.14/library/inspect.html) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*