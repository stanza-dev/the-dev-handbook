---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-understanding-metaclasses"
---

# Metaclasses

In Python, classes are objects too. A metaclass is the class of a class. The default metaclass is `type`.

## Customizing Class Creation
You can control how a class is created by defining a custom metaclass. This is how ORMs (like Django or SQLAlchemy) work.

```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=Singleton):
    pass
```

## Code Examples

**Dynamic Class Creation**

```python
# Using 'type' dynamically to create a class
MyClass = type('MyClass', (object,), {'x': 5})

obj = MyClass()
print(obj.x) # 5
```


---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*