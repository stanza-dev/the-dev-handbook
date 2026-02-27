---
source_course: "python"
source_lesson: "python-class-basics"
---

# Class Fundamentals

## Introduction
Classes are the foundation of object-oriented programming in Python. They let you bundle data (attributes) and behavior (methods) into reusable blueprints. This lesson covers class definitions, instance vs class attributes, and the special "dunder" methods that integrate your objects with Python's built-in operations.

## Key Concepts
- **Class**: A blueprint for creating objects, defined with the `class` keyword.
- **Instance attribute**: Data unique to each object, typically set in `__init__`.
- **Class attribute**: Data shared across all instances of a class.
- **Dunder methods**: Special methods like `__init__`, `__str__`, `__repr__`, and `__eq__` that hook into Python operators and built-in functions.

## Real World Context
Almost every Python library and framework uses classes. Django models, SQLAlchemy tables, Pydantic schemas, and pytest fixtures are all class-based. Understanding how `__init__`, `__repr__`, and operator overloading work lets you build objects that integrate seamlessly with print(), sorted(), ==, and the rest of Python's ecosystem.

## Deep Dive

### Defining a Class

```python
class Dog:
    # Class attribute (shared by all instances)
    species = "Canis familiaris"
    
    def __init__(self, name, age):
        # Instance attributes
        self.name = name
        self.age = age
    
    def bark(self):
        return f"{self.name} says woof!"
    
    def __str__(self):
        return f"{self.name}, {self.age} years old"
```

### Creating Objects

```python
my_dog = Dog("Buddy", 3)
print(my_dog.name)    # "Buddy"
print(my_dog.bark())  # "Buddy says woof!"
print(Dog.species)    # "Canis familiaris"
```

### Instance vs Class Attributes

```python
class Counter:
    count = 0  # Class attribute
    
    def __init__(self):
        Counter.count += 1
        self.id = Counter.count  # Instance attribute

c1 = Counter()
c2 = Counter()
print(Counter.count)  # 2
print(c1.id, c2.id)   # 1, 2
```

### Special Methods (Dunder Methods)

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"
    
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    
    def __len__(self):
        return int((self.x**2 + self.y**2)**0.5)
```

## Common Pitfalls
1. **Using a mutable class attribute unintentionally** -- If you define `class Foo: items = []`, all instances share that list. Mutating it via one instance affects all others. Use `self.items = []` in `__init__` for per-instance data.
2. **Forgetting `self` in method definitions** -- Omitting `self` as the first parameter of an instance method causes a confusing TypeError when the method is called.
3. **Confusing `__str__` and `__repr__`** -- `__str__` is for end-user display (print, f-strings); `__repr__` is for developer debugging (REPL, logging). Implement both, and make `__repr__` unambiguous.

## Best Practices
1. **Always implement `__repr__`** -- A good `__repr__` makes debugging dramatically easier. Aim for output that could recreate the object: `Vector(3, 4)`.
2. **Keep `__init__` focused on initialization** -- Avoid heavy computation or I/O in `__init__`. Use factory methods or classmethod constructors for complex setup.

## Summary
- Classes bundle state (attributes) and behavior (methods) into reusable blueprints.
- Instance attributes are unique per object; class attributes are shared.
- Dunder methods (`__init__`, `__repr__`, `__eq__`, `__add__`) integrate objects with Python operators and built-ins.
- Avoid mutable class attributes for per-instance data.
- Always implement `__repr__` for debuggable objects.

## Code Examples

**Common dunder methods**

```python
# Common dunder methods
class MyClass:
    def __init__(self):     # Constructor
        pass
    def __str__(self):      # str(obj), print(obj)
        return "human readable"
    def __repr__(self):     # repr(obj), debugging
        return "MyClass()"
    def __len__(self):      # len(obj)
        return 0
    def __bool__(self):     # bool(obj), if obj:
        return True
    def __iter__(self):     # for x in obj:
        return iter([])
    def __getitem__(self, key):  # obj[key]
        pass
```


## Resources

- [Classes](https://docs.python.org/3.14/tutorial/classes.html) — Official Python 3.14 tutorial on classes, instances, and object-oriented programming

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*