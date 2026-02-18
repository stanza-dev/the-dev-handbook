---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-class-decorators"
---

# Decorating Classes

## Class as Decorator

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

## Decorating a Class

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

## Class Decorator to Add Methods

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


---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*