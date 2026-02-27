---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-class-decorators"
---

# Class Decorators

## Introduction

Decorators are not limited to functions -- they can also be classes that act as decorators, or functions that decorate classes. This dual capability opens up powerful patterns: using a class with `__call__` to maintain state across invocations, or modifying a class by injecting methods, enforcing constraints, or registering it in a global registry.

## Key Concepts

- **Class as a decorator** -- A class with `__init__` (to receive the decorated function) and `__call__` (to execute when the decorated function is called). This lets the decorator maintain per-function state like call counts or caches.
- **Decorating a class** -- A function that takes a class as its argument, modifies or wraps it, and returns the result. This is identical in syntax to function decoration but operates on a class object.
- **`functools.update_wrapper`** -- The function-based equivalent of `@functools.wraps`, used in class-based decorators to copy metadata from the original function onto the class instance.

## Real World Context

Class-based decorators shine when you need state. A production rate limiter might track timestamps of recent calls; a circuit breaker might count consecutive failures and switch to an open state. These are awkward to implement with closures but natural with class attributes. On the other side, decorating classes is the backbone of patterns like `@dataclass`, `@singleton`, plugin registration systems, and ORM model registration. Django's `@admin.register(MyModel)` is a class decorator that registers a model with the admin site.

## Deep Dive

When a class implements `__init__` and `__call__`, it can serve as a decorator. The `__init__` method receives the decorated function, and `__call__` is invoked each time the decorated function is called. This pattern lets you store state as instance attributes:

```python
class CountCalls:
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.count = 0
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"Call #{self.count}")
        return self.func(*args, **kwargs)

@CountCalls
def greet(name):
    return f"Hello, {name}"

greet("Alice")  # Call #1
greet("Bob")    # Call #2
print(greet.count)  # 2
```

After decoration, `greet` is an instance of `CountCalls`. Calling `greet("Alice")` invokes `CountCalls.__call__`, which increments the counter and delegates to the original function. The `functools.update_wrapper(self, func)` call copies `__name__`, `__doc__`, and other metadata from the original function onto the class instance.

Decorators can also target classes themselves. A classic example is the singleton pattern, which ensures only one instance of a class ever exists:

```python
def singleton(cls):
    instances = {}
    
    @functools.wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class Database:
    def __init__(self):
        print("Connecting...")

db1 = Database()  # Connecting...
db2 = Database()  # No output (returns same instance)
print(db1 is db2)  # True
```

Here, the `singleton` decorator replaces the `Database` class with a `get_instance` function that caches the first instance and returns it on subsequent calls. The `@functools.wraps(cls)` ensures the wrapper retains the class's name and docstring.

Class decorators can also modify a class in place by adding or replacing methods. This is how `@dataclass` works under the hood -- it inspects annotations and generates `__init__`, `__repr__`, `__eq__`, and other methods:

```python
def add_repr(cls):
    def __repr__(self):
        attrs = ", ".join(f"{k}={v!r}" for k, v in self.__dict__.items())
        return f"{cls.__name__}({attrs})"
    
    cls.__repr__ = __repr__
    return cls

@add_repr
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

print(Point(1, 2))  # Point(x=1, y=2)
```

The `add_repr` decorator attaches a new `__repr__` method to `Point` and returns the modified class. Unlike the singleton example, this approach modifies the class in place rather than replacing it, which preserves `isinstance` checks and class identity.

## Common Pitfalls

- **Forgetting `functools.update_wrapper` in class-based decorators.** Without it, the decorated function's `__name__` and `__doc__` will reflect the class, not the original function. Use `functools.update_wrapper(self, func)` in `__init__`.
- **Breaking `isinstance` checks with wrapping decorators.** The singleton example replaces the class with a function, so `isinstance(db1, Database)` may behave unexpectedly. If `isinstance` checks are important, modify the class in place rather than replacing it.
- **Not returning the class from a class decorator.** If your decorator modifies a class in place but forgets to `return cls`, the decorated class becomes `None`, causing immediate and confusing errors.

## Best Practices

- Prefer class-based decorators when you need to maintain state (call counts, timing data, caches) across invocations of the decorated function.
- When decorating a class, prefer in-place modification (adding methods, setting attributes) over wrapping it in a function, to preserve `isinstance` and `issubclass` compatibility.
- Always use `functools.update_wrapper(self, func)` in class-based decorators and `@functools.wraps(cls)` in class-decorating functions to maintain metadata.

## Summary

- A class with `__init__` and `__call__` can serve as a decorator, letting you maintain per-function state as instance attributes.
- Functions can decorate classes by modifying them in place (adding methods) or wrapping them (singleton pattern).
- Use `functools.update_wrapper` in class-based decorators to preserve the original function's metadata.
- Prefer in-place class modification over wrapping to maintain `isinstance` compatibility.
- Class decorators are the foundation of patterns like `@dataclass`, `@singleton`, and plugin registries.

## Code Examples

**Custom dataclass decorator**

```python
def dataclass_like(cls):
    """Simple dataclass-like decorator."""
    # Get annotations
    annotations = getattr(cls, '__annotations__', {})
    
    # Generate __init__
    def __init__(self, **kwargs):
        for name, type_ in annotations.items():
            setattr(self, name, kwargs.get(name))
    
    # Generate __repr__
    def __repr__(self):
        attrs = ', '.join(f"{k}={getattr(self, k)!r}" 
                         for k in annotations)
        return f"{cls.__name__}({attrs})"
    
    cls.__init__ = __init__
    cls.__repr__ = __repr__
    return cls

@dataclass_like
class User:
    name: str
    age: int

u = User(name="Alice", age=30)
print(u)  # User(name='Alice', age=30)
```


## Resources

- [PEP 3129 — Class Decorators](https://peps.python.org/pep-3129/) — The PEP that introduced class decorators
- [dataclasses — Data Classes](https://docs.python.org/3.14/library/dataclasses.html) — Official docs for @dataclass, the most common class decorator

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*