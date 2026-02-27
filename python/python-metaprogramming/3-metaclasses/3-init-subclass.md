---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-init-subclass"
---

# __init_subclass__ as a Lighter Alternative

## Introduction

Python 3.6 introduced `__init_subclass__`, a class method hook that runs whenever a class is subclassed. It provides most of the power of metaclasses for the common use case of customizing subclass creation -- without the complexity of writing a full metaclass. If your goal is to validate, register, or modify subclasses, `__init_subclass__` should be your first choice.

## Key Concepts

- **`__init_subclass__(cls, **kwargs)`**: A classmethod called on the *parent* class whenever a new subclass is defined. The subclass is already created when this runs.
- **Keyword arguments**: Extra keyword arguments from the `class` statement are forwarded to `__init_subclass__`.
- **No metaclass needed**: Works with the default `type` metaclass, avoiding metaclass conflicts entirely.
- **Composable**: Multiple classes in an inheritance chain can each define `__init_subclass__` and they will all run via `super()`.

## Real World Context

- **Plugin registration**: Libraries use `__init_subclass__` to auto-register plugins when they are defined, without requiring a separate registration step.
- **Pydantic v2**: Moved from metaclasses to `__init_subclass__` and `__class_getitem__` for model configuration.
- **dataclasses**: The `@dataclass` decorator could theoretically be replaced with `__init_subclass__` in a base class.
- **Validation frameworks**: Enforce that subclasses define required attributes or follow naming conventions.

## Deep Dive

Basic usage -- auto-registering subclasses:

```python
class Plugin:
    _plugins = {}

    def __init_subclass__(cls, *, plugin_name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        name = plugin_name or cls.__name__.lower()
        Plugin._plugins[name] = cls
        print(f"Registered plugin: {name}")

class AudioPlugin(Plugin, plugin_name="audio"):
    pass
# Registered plugin: audio

class VideoPlugin(Plugin):
    pass
# Registered plugin: videoplugin

print(Plugin._plugins)
# {'audio': <class 'AudioPlugin'>, 'videoplugin': <class 'VideoPlugin'>}
```

Enforcing that subclasses implement required methods:

```python
class Serializer:
    _required = ['serialize', 'deserialize']

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        for method in cls._required:
            if not callable(getattr(cls, method, None)):
                raise TypeError(
                    f"{cls.__name__} must implement {method}()"
                )

class JSONSerializer(Serializer):
    def serialize(self, data):
        import json
        return json.dumps(data)

    def deserialize(self, text):
        import json
        return json.loads(text)

# This raises TypeError:
# class BrokenSerializer(Serializer):
#     def serialize(self, data): ...
#     # Missing deserialize!
```

Compare the same pattern with a metaclass:

```python
# With metaclass (more complex)
class PluginMeta(type):
    _plugins = {}
    def __new__(mcs, name, bases, namespace, **kwargs):
        cls = super().__new__(mcs, name, bases, namespace)
        if bases:
            mcs._plugins[name] = cls
        return cls

class Plugin(metaclass=PluginMeta):
    pass

# With __init_subclass__ (simpler, no metaclass)
class Plugin:
    _plugins = {}
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        Plugin._plugins[cls.__name__] = cls
```

## When to Use Each Approach

| Need | Use |
|------|-----|
| Register/validate subclasses | `__init_subclass__` |
| Accept keyword args on class statement | `__init_subclass__` |
| Modify the class namespace before body executes | Metaclass (`__prepare__`) |
| Control class object creation itself | Metaclass (`__new__`) |
| Customize instance creation (singleton, pool) | Metaclass (`__call__`) |
| Need to work with classes that already have a metaclass | `__init_subclass__` |

## Common Pitfalls

- **Forgetting `super().__init_subclass__(**kwargs)`**: If you omit the `super()` call, parent classes in the MRO that also define `__init_subclass__` will be silently skipped. Always call `super()`.
- **Forgetting `**kwargs`**: Both the parameter list and the `super()` call must include `**kwargs` to forward unrecognized keyword arguments up the chain.
- **Trying to modify the class before it exists**: `__init_subclass__` runs *after* the class is created. If you need to change the namespace during class body execution, you need a metaclass with `__prepare__`.
- **Assuming it runs on the defining class**: `__init_subclass__` only runs on *subclasses*, not on the class where it is defined.

## Best Practices

- Default to `__init_subclass__` for class customization -- it covers 90% of use cases with far less complexity.
- Always accept and forward `**kwargs` to maintain compatibility with cooperative multiple inheritance.
- Use keyword arguments in the class statement for clean, declarative configuration (`class MyPlugin(Plugin, name="audio")`).
- Reserve metaclasses for truly advanced needs: custom namespaces, controlling class object allocation, or instance creation hooks.

## Summary

`__init_subclass__` is a Python 3.6+ hook that lets a parent class customize its subclasses without a metaclass. It supports keyword arguments from the class statement, composes well with multiple inheritance, and avoids metaclass conflicts. Use it as your default tool for subclass validation, registration, and configuration. Reach for a full metaclass only when you need to control the class namespace (`__prepare__`) or the class/instance creation process itself (`__new__`/`__call__`).

## Code Examples

**Plugin registration with __init_subclass__**

```python
class Plugin:
    """Base class that auto-registers all subclasses."""
    _plugins = {}

    def __init_subclass__(cls, *, plugin_name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        name = plugin_name or cls.__name__.lower()
        Plugin._plugins[name] = cls

class AudioPlugin(Plugin, plugin_name="audio"):
    def process(self):
        return "Processing audio..."

class VideoPlugin(Plugin, plugin_name="video"):
    def process(self):
        return "Processing video..."

print(Plugin._plugins)
# {'audio': <class 'AudioPlugin'>, 'video': <class 'VideoPlugin'>}

# Instantiate plugin by name
plugin = Plugin._plugins['audio']()
print(plugin.process())  # Processing audio...
```

**Enforcing required attributes**

```python
class Validated:
    """Base class that enforces required class attributes."""
    _required_attrs = []

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        for attr in cls._required_attrs:
            if not hasattr(cls, attr):
                raise TypeError(
                    f"{cls.__name__} must define '{attr}'"
                )

class Animal(Validated):
    _required_attrs = ['species', 'sound']

class Dog(Animal):
    species = 'Canis familiaris'
    sound = 'Woof'

# class Cat(Animal):  # TypeError: Cat must define 'sound'
#     species = 'Felis catus'
```


## Resources

- [PEP 487 -- Simpler customisation of class creation](https://peps.python.org/pep-0487/) — undefined
- [Python docs -- __init_subclass__](https://docs.python.org/3/reference/datamodel.html#object.__init_subclass__) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*