---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-dynamic-attributes"
---

# getattr, setattr & the MRO

## Introduction

Python treats attributes as fully dynamic -- you can read, write, check, and delete them at runtime using just a string name. Combined with the Method Resolution Order (MRO), which governs how Python searches through class hierarchies, dynamic attribute access becomes one of the most powerful tools in the language. Understanding these mechanics is essential for anyone writing frameworks, ORMs, or plugin systems.

## Key Concepts

- **`getattr(obj, name, default)`** -- Reads an attribute by name, with an optional fallback value if the attribute does not exist.
- **`setattr(obj, name, value)`** -- Sets an attribute by name, creating it if it does not already exist.
- **`hasattr(obj, name)`** -- Returns `True` if the attribute exists (internally calls `getattr` and checks for `AttributeError`).
- **`delattr(obj, name)`** -- Deletes an attribute by name.
- **Method Resolution Order (MRO)** -- The order in which Python searches base classes when looking up an attribute, computed using the C3 linearization algorithm.
- **`__getattr__`** -- A hook called only when normal attribute lookup fails.
- **`__getattribute__`** -- A hook called on every attribute access, before any other lookup.

## Real World Context

Consider an ORM like Django's model layer. When you write `user.email`, Django does not simply look up a dictionary key -- it walks through descriptors, class attributes, and the MRO to resolve the field, potentially triggering database queries or lazy loading. Understanding the attribute lookup chain is what separates someone who uses an ORM from someone who can debug or extend one.

## Deep Dive

Python provides four built-in functions for working with attributes dynamically. Together, they let you treat any object as a flexible key-value store where the keys are strings.

```python
class Config:
    debug = False
    version = "1.0"

cfg = Config()

# getattr — read an attribute by name
print(getattr(cfg, 'debug'))          # False
print(getattr(cfg, 'missing', None))  # None (default)

# setattr — set an attribute by name
setattr(cfg, 'debug', True)
print(cfg.debug)  # True

# hasattr — check existence (calls getattr internally)
print(hasattr(cfg, 'version'))  # True
print(hasattr(cfg, 'missing'))  # False

# delattr — delete an attribute
setattr(cfg, 'temp', 42)
delattr(cfg, 'temp')
print(hasattr(cfg, 'temp'))  # False
```

These four functions are the foundation for dynamic dispatch patterns, where you look up and call methods by name at runtime. Here is a practical example of a command handler that routes commands to methods dynamically.

```python
class CommandHandler:
    def cmd_start(self):
        return "Starting..."

    def cmd_stop(self):
        return "Stopping..."

    def dispatch(self, command: str):
        method = getattr(self, f'cmd_{command}', None)
        if method is None:
            return f"Unknown command: {command}"
        return method()

handler = CommandHandler()
print(handler.dispatch('start'))  # Starting...
print(handler.dispatch('quit'))   # Unknown command: quit
```

The `dispatch` method uses `getattr` with a default of `None` to safely look up handler methods by constructing the method name from user input. This is a clean alternative to long `if/elif` chains.

When you access `obj.attr`, Python follows a specific lookup chain with four steps. Understanding this order is critical for predicting how attribute resolution behaves in complex class hierarchies.

1. **Data descriptors** on the class (e.g., `property`, descriptors with `__set__`)
2. **Instance `__dict__`** -- the object's own attributes
3. **Non-data descriptors** and class attributes -- walked via the MRO
4. **`__getattr__`** -- called only if all above fail

The following example demonstrates steps 2, 3, and 4 in action.

```python
class Fallback:
    x = 10  # Class attribute (step 3)

    def __getattr__(self, name):
        # Step 4: only called when normal lookup fails
        return f"{name} not found"

obj = Fallback()
obj.y = 20  # Instance attribute (step 2)

print(obj.x)  # 10 — from class (step 3)
print(obj.y)  # 20 — from instance (step 2)
print(obj.z)  # 'z not found' — from __getattr__ (step 4)
```

Accessing `obj.x` finds `x` as a class attribute (step 3). Accessing `obj.y` finds it in the instance dictionary (step 2, which takes priority over step 3). Accessing `obj.z` triggers `__getattr__` because `z` is not found anywhere else.

The Method Resolution Order defines the order Python searches base classes when looking up methods or attributes. Python uses the **C3 linearization** algorithm, which guarantees a consistent and predictable ordering even in complex diamond inheritance scenarios.

```python
class A:
    def greet(self):
        return "A"

class B(A):
    def greet(self):
        return "B"

class C(A):
    def greet(self):
        return "C"

class D(B, C):
    pass

print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

print(D().greet())  # 'B' — B comes before C in the MRO
```

Because `D` inherits from `B` first, and `B` defines `greet`, the MRO resolves the method to `B.greet` without ever checking `C` or `A`.

You can inspect the MRO using three equivalent approaches. All three return the same ordering, just in slightly different formats.

```python
import inspect

# Three ways to see the MRO:
print(D.__mro__)            # Tuple of classes
print(D.mro())              # List (calls type.mro())
print(inspect.getmro(D))    # Tuple (via inspect)
```

Each approach is useful in different contexts -- `__mro__` is the most direct, `mro()` is a method you can override, and `inspect.getmro()` is the safest for introspection tooling.

Finally, it is important to understand the difference between `__getattr__` and `__getattribute__`. They serve very different purposes and misusing `__getattribute__` is a common source of infinite recursion bugs.

```python
class Logged:
    def __init__(self):
        self.x = 10

    def __getattribute__(self, name):
        print(f"Accessing {name}")  # Fires for EVERY access
        return super().__getattribute__(name)

    def __getattr__(self, name):
        print(f"{name} not found")  # Only fires when missing
        raise AttributeError(name)

obj = Logged()
obj.x       # Accessing x → 10
obj.y       # Accessing y → y not found → AttributeError
```

`__getattribute__` intercepts every single attribute access, including successful ones. `__getattr__` is only invoked as a last resort when the normal lookup chain fails.

## Common Pitfalls

- **Infinite recursion in `__getattribute__`** -- If you access `self.anything` inside `__getattribute__`, it triggers another call to `__getattribute__`. Always use `super().__getattribute__(name)` or `object.__getattribute__(self, name)` to break the cycle.
- **`hasattr` swallowing exceptions** -- `hasattr` works by calling `getattr` and catching `AttributeError`. If your `__getattr__` raises a different exception (like `ValueError`), `hasattr` will propagate it in Python 3, which can cause unexpected crashes.
- **Forgetting the MRO in multiple inheritance** -- When two parent classes define the same method, the MRO determines which one wins. If you do not understand C3 linearization, you may get a different method than expected.

## Best Practices

- **Always provide a default with `getattr` when the attribute might not exist** -- Using `getattr(obj, 'name', None)` is safer and cleaner than wrapping the access in a `try/except AttributeError` block.
- **Use `__getattr__` instead of `__getattribute__`** -- Unless you genuinely need to intercept every attribute access (e.g., for proxying or logging), stick with `__getattr__` which only fires on misses.
- **Check the MRO when debugging inheritance issues** -- Print `MyClass.__mro__` to see exactly which classes Python will search and in what order.

## Summary

- Python's four attribute functions (`getattr`, `setattr`, `hasattr`, `delattr`) let you manipulate attributes dynamically using string names.
- The attribute lookup chain follows a strict order: data descriptors, instance dict, class attributes via MRO, and finally `__getattr__`.
- The MRO uses C3 linearization to determine the order Python searches base classes in multiple inheritance.
- `__getattr__` is a safe fallback hook; `__getattribute__` intercepts all access and requires careful use of `super()` to avoid infinite recursion.
- Dynamic dispatch via `getattr` is a clean, Pythonic alternative to long conditional chains.

## Code Examples

**Dynamic configuration with getattr/setattr**

```python
class DynamicConfig:
    """Configuration that loads defaults from a dict."""
    _defaults = {
        'timeout': 30,
        'retries': 3,
        'debug': False,
    }
    
    def __getattr__(self, name):
        if name in self._defaults:
            return self._defaults[name]
        raise AttributeError(f"No setting '{name}'")
    
    def set(self, name, value):
        setattr(self, name, value)

cfg = DynamicConfig()
print(cfg.timeout)   # 30 (from _defaults via __getattr__)
cfg.set('timeout', 60)
print(cfg.timeout)   # 60 (from instance __dict__, bypasses __getattr__)
```

**MRO and attribute resolution in diamond inheritance**

```python
class A:
    value = 'A'

class B(A):
    pass

class C(A):
    value = 'C'

class D(B, C):
    pass

# MRO determines which 'value' is found first
print(D.__mro__)
# (D, B, C, A, object)
print(D.value)  # 'C' — B has no value, so C (next in MRO) wins
```


## Resources

- [inspect — Inspect live objects (Python 3.14)](https://docs.python.org/3.14/library/inspect.html) — undefined
- [Built-in Functions — getattr, setattr, hasattr, delattr](https://docs.python.org/3.14/library/functions.html#getattr) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*