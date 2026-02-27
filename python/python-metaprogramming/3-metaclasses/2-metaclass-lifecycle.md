---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-metaclass-lifecycle"
---

# __new__ vs __init__ vs __call__ in Metaclasses

## Introduction

A metaclass has three key methods that form its lifecycle: `__new__`, `__init__`, and `__call__`. Each fires at a different stage of class creation and instantiation. Confusing them is the number one source of metaclass bugs. This lesson clarifies when each method runs and what it controls.

## Key Concepts

- **`__new__(mcs, name, bases, namespace)`**: Creates and returns the new class object. Runs once when the `class` statement is executed. This is where you can modify the class before it exists.
- **`__init__(cls, name, bases, namespace)`**: Initializes the already-created class object. Runs after `__new__`. Use this for setup that does not need to change the class structure.
- **`__call__(cls, *args, **kwargs)`**: Runs every time you instantiate the class (e.g., `obj = MyClass()`). It orchestrates the instance's `__new__` and `__init__`.
- **`__prepare__(mcs, name, bases)`**: Returns the namespace dict for the class body. Runs before `__new__`. Useful for ordered or restricted namespaces.

## Real World Context

- **Django REST Framework serializers** use metaclass `__new__` to collect field definitions from the class body and store them in a `_declared_fields` attribute before the class is fully created.
- **Enum** uses metaclass `__new__` to convert class attributes into enum members and prevent duplicate values.
- **Singleton pattern** is implemented by overriding `__call__` on the metaclass to cache and reuse the single instance.

## Deep Dive

Here is the full lifecycle with print statements to show the execution order:

```python
class TracingMeta(type):
    def __prepare__(mcs, name, bases, **kwargs):
        print(f"1. __prepare__: creating namespace for {name}")
        return super().__prepare__(name, bases, **kwargs)

    def __new__(mcs, name, bases, namespace, **kwargs):
        print(f"2. __new__: creating class object '{name}'")
        cls = super().__new__(mcs, name, bases, namespace)
        return cls

    def __init__(cls, name, bases, namespace, **kwargs):
        print(f"3. __init__: initializing class '{name}'")
        super().__init__(name, bases, namespace)

    def __call__(cls, *args, **kwargs):
        print(f"4. __call__: creating instance of '{cls.__name__}'")
        instance = super().__call__(*args, **kwargs)
        return instance

class MyClass(metaclass=TracingMeta):
    pass
# Output during class definition:
# 1. __prepare__: creating namespace for MyClass
# 2. __new__: creating class object 'MyClass'
# 3. __init__: initializing class 'MyClass'

obj = MyClass()
# Output during instantiation:
# 4. __call__: creating instance of 'MyClass'
```

Notice that `__prepare__`, `__new__`, and `__init__` run at **class definition time** (when the `class` statement executes). Only `__call__` runs at **instantiation time** (when you call `MyClass()`).

Here is a practical use of `__new__` to validate class definitions:

```python
class InterfaceMeta(type):
    """Metaclass that ensures subclasses implement required methods."""
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)

        # Skip the base interface class itself
        if bases:
            required = getattr(cls, '_required_methods', [])
            for method_name in required:
                if method_name not in namespace:
                    raise TypeError(
                        f"Class '{name}' must implement '{method_name}'"
                    )
        return cls

class Serializer(metaclass=InterfaceMeta):
    _required_methods = ['serialize', 'deserialize']

class JSONSerializer(Serializer):
    def serialize(self, data):
        import json
        return json.dumps(data)

    def deserialize(self, text):
        import json
        return json.loads(text)

# This would raise TypeError:
# class BrokenSerializer(Serializer):
#     def serialize(self, data): ...
#     # Missing deserialize!
```

And here is `__call__` for the Singleton pattern:

```python
class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=Singleton):
    def __init__(self):
        print("Connecting to database...")

db1 = Database()  # Connecting to database...
db2 = Database()  # No output -- returns cached instance
print(db1 is db2) # True
```

## Common Pitfalls

- **Modifying the class in `__init__` instead of `__new__`**: In `__init__`, the class already exists. If you need to alter the class structure (e.g., add/remove attributes, change bases), do it in `__new__` where you control what gets created.
- **Forgetting to call `super()`**: Always call `super().__new__()`, `super().__init__()`, or `super().__call__()` -- otherwise the default behavior (like actually creating the class or instance) is skipped entirely.
- **Confusing metaclass `__call__` with class `__call__`**: `Metaclass.__call__` runs when you call the *class* (to make an instance). `Class.__call__` runs when you call an *instance*. They live at different levels.
- **Using `__new__` when `__init__` suffices**: If you only need to record metadata or register the class (not alter its structure), prefer `__init__` -- it is simpler.

## Best Practices

- Use `__new__` to modify the class before it is created (validate methods, alter namespace, inject attributes).
- Use `__init__` for post-creation setup that does not alter the class structure (registration, logging).
- Use `__call__` to customize instance creation (caching, pooling, singletons).
- Add tracing (like the `TracingMeta` example) when debugging metaclass issues -- it reveals the exact execution order.

## Summary

The metaclass lifecycle consists of `__prepare__` (namespace creation), `__new__` (class object creation), `__init__` (class initialization), and `__call__` (instance creation). The first three run at class definition time; `__call__` runs each time you instantiate the class. Choosing the right method depends on *when* you need to intervene: before the class exists (`__new__`), after it exists (`__init__`), or when instances are created (`__call__`).

## Code Examples

**Full metaclass lifecycle tracing**

```python
class TracingMeta(type):
    def __prepare__(mcs, name, bases, **kwargs):
        print(f"1. __prepare__: creating namespace for {name}")
        return super().__prepare__(name, bases, **kwargs)

    def __new__(mcs, name, bases, namespace, **kwargs):
        print(f"2. __new__: creating class object '{name}'")
        return super().__new__(mcs, name, bases, namespace)

    def __init__(cls, name, bases, namespace, **kwargs):
        print(f"3. __init__: initializing class '{name}'")
        super().__init__(name, bases, namespace)

    def __call__(cls, *args, **kwargs):
        print(f"4. __call__: creating instance of '{cls.__name__}'")
        return super().__call__(*args, **kwargs)

class MyClass(metaclass=TracingMeta):
    pass
# 1. __prepare__: creating namespace for MyClass
# 2. __new__: creating class object 'MyClass'
# 3. __init__: initializing class 'MyClass'

obj = MyClass()
# 4. __call__: creating instance of 'MyClass'
```

**Singleton via metaclass __call__**

```python
class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Config(metaclass=Singleton):
    def __init__(self):
        self.settings = {}

c1 = Config()
c2 = Config()
print(c1 is c2)  # True
```


## Resources

- [Python Data Model -- Customizing class creation](https://docs.python.org/3/reference/datamodel.html#customizing-class-creation) — undefined
- [Stack Overflow -- What are metaclasses in Python?](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*