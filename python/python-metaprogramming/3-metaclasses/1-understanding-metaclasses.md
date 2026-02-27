---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-understanding-metaclasses"
---

# Understanding Metaclasses

## Introduction

In Python, everything is an object -- including classes themselves. A **metaclass** is the class of a class: it defines how classes are created and behave. The default metaclass for every class is `type`. Understanding metaclasses unlocks the ability to customize class creation, enforce coding standards, and build powerful frameworks.

## Key Concepts

- **Classes are objects**: When Python encounters a `class` statement, it creates a class *object* using its metaclass.
- **`type` is the default metaclass**: `type('Foo', (object,), {'x': 1})` creates a class dynamically.
- **`metaclass=` keyword**: Pass a custom metaclass in the class definition to control class creation.
- **`__prepare__`**: An optional metaclass method that returns the namespace dictionary used during class body execution.

## Real World Context

Metaclasses power some of the most popular Python frameworks:
- **Django ORM**: `models.Model` uses a metaclass to collect field definitions and build database queries.
- **SQLAlchemy**: Declarative base uses metaclasses to map Python classes to database tables.
- **Abstract Base Classes (ABC)**: `ABCMeta` ensures abstract methods are implemented in subclasses.
- **Pydantic v1**: Used metaclasses for model field registration (v2 moved to `__init_subclass__`).

## Deep Dive

When you write a `class` statement, Python executes these steps internally:

```python
# What you write:
class Foo(Base, metaclass=Meta):
    x = 10
    def greet(self):
        return "hello"

# What Python does (simplified):
# 1. Call Meta.__prepare__('Foo', (Base,)) to get namespace dict
# 2. Execute class body inside that namespace
# 3. Call Meta('Foo', (Base,), namespace) to create the class
```

Here is a practical metaclass that enforces all methods have docstrings:

```python
class DocstringEnforcer(type):
    def __new__(mcs, name, bases, namespace):
        for attr_name, attr_value in namespace.items():
            if callable(attr_value) and not attr_name.startswith('_'):
                if not attr_value.__doc__:
                    raise TypeError(
                        f"Method '{attr_name}' in '{name}' "
                        f"must have a docstring"
                    )
        return super().__new__(mcs, name, bases, namespace)

class APIEndpoint(metaclass=DocstringEnforcer):
    def get(self):
        """Handle GET request."""
        pass

    def post(self):
        """Handle POST request."""
        pass
```

You can also use `type` directly to create classes dynamically:

```python
# Create a class at runtime -- no class statement needed
MyClass = type('MyClass', (object,), {'x': 5, 'hello': lambda self: 'hi'})

obj = MyClass()
print(obj.x)        # 5
print(obj.hello())   # hi
print(type(MyClass)) # <class 'type'>
```

## Common Pitfalls

- **Overusing metaclasses**: Most problems that seem to need a metaclass can be solved with class decorators or `__init_subclass__`. Use metaclasses only when you need to control the *creation* of the class object itself.
- **Metaclass conflicts**: If two parent classes use different metaclasses, Python raises a `TypeError`. The fix is to create a new metaclass that inherits from both.
- **Confusing `type` as a function vs. metaclass**: `type(obj)` returns the type of an object, but `type(name, bases, dict)` creates a new class. Same name, different behavior based on argument count.

## Best Practices

- Prefer class decorators or `__init_subclass__` over metaclasses when possible.
- Keep metaclass logic focused on class creation concerns, not instance behavior.
- Document your metaclass thoroughly -- they are hard to debug.
- Use `super().__new__()` and `super().__init__()` to stay compatible with metaclass inheritance.

## Summary

Metaclasses are the mechanism behind class creation in Python. The default metaclass `type` handles normal class creation, but custom metaclasses let you intercept and modify how classes are built. They are powerful but should be used sparingly -- frameworks like Django and SQLAlchemy use them so that application developers do not have to.

## Code Examples

**Dynamic Class Creation with type**

```python
# Using 'type' dynamically to create a class
MyClass = type('MyClass', (object,), {'x': 5})

obj = MyClass()
print(obj.x)        # 5
print(type(MyClass)) # <class 'type'>
```

**Auto-registering metaclass**

```python
class RegistryMeta(type):
    """Metaclass that auto-registers all subclasses."""
    _registry = {}

    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if bases:  # Don't register the base class itself
            mcs._registry[name] = cls
        return cls

class Plugin(metaclass=RegistryMeta):
    pass

class AudioPlugin(Plugin):
    pass

class VideoPlugin(Plugin):
    pass

print(RegistryMeta._registry)
# {'AudioPlugin': <class 'AudioPlugin'>, 'VideoPlugin': <class 'VideoPlugin'>}
```


## Resources

- [Python Data Model -- Metaclasses](https://docs.python.org/3/reference/datamodel.html#metaclasses) — undefined
- [Real Python -- Python Metaclasses](https://realpython.com/python-metaclasses/) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*