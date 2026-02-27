---
source_course: "python"
source_lesson: "python-inheritance-composition"
---

# Inheritance & Composition

## Introduction
Inheritance lets classes share behavior through parent-child relationships, while composition assembles behavior by holding references to other objects. This lesson covers basic inheritance, `super()`, multiple inheritance with the MRO, and the principle of composition over inheritance.

## Key Concepts
- **Inheritance**: A child class receives all attributes and methods from its parent class.
- **`super()`**: Calls the parent class's implementation, essential for cooperative multiple inheritance.
- **MRO (Method Resolution Order)**: The order Python searches through a class hierarchy when looking up a method, computed via C3 linearization.
- **Composition**: Holding references to other objects instead of inheriting from them -- preferred for "has-a" relationships.

## Real World Context
Django's class-based views, SQLAlchemy's declarative models, and Python's own `io` module all use inheritance extensively. However, over-deep hierarchies become brittle. The industry consensus -- and the advice in *Design Patterns* -- is to favor composition for flexibility. Mixins (small, focused base classes) offer a pragmatic middle ground used by libraries like Django REST Framework.

## Deep Dive

### Basic Inheritance

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        raise NotImplementedError

class Dog(Animal):
    def speak(self):
        return f"{self.name} barks"

class Cat(Animal):
    def speak(self):
        return f"{self.name} meows"
```

### Using super()

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

class Manager(Employee):
    def __init__(self, name, salary, department):
        super().__init__(name, salary)  # Call parent __init__
        self.department = department
```

### Multiple Inheritance & MRO

```python
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):  # Multiple inheritance
    pass

# Method Resolution Order
print(D.__mro__)  # D -> B -> C -> A -> object
d = D()
print(d.method())  # "B" (leftmost parent first)
```

### Composition Over Inheritance

```python
# Prefer composition when "has-a" relationship makes sense
class Engine:
    def start(self):
        return "Engine started"

class Car:
    def __init__(self):
        self.engine = Engine()  # Composition
    
    def start(self):
        return self.engine.start()
```

## Common Pitfalls
1. **Forgetting to call `super().__init__()`** -- If a child class overrides `__init__` without calling `super().__init__()`, parent attributes are never set, leading to `AttributeError` at runtime.
2. **Deep inheritance hierarchies** -- More than two or three levels of inheritance becomes hard to reason about and debug. Flatten hierarchies using mixins or composition.
3. **Diamond inheritance surprises** -- With multiple inheritance, the MRO may call methods in an order you did not expect. Always check `ClassName.__mro__` and use `super()` consistently.

## Best Practices
1. **Favor composition for "has-a" relationships** -- A `Car` has an `Engine`; it does not inherit from one. Composition is more flexible and easier to test.
2. **Use mixins for cross-cutting behavior** -- Small, focused base classes like `JSONMixin` or `LoggingMixin` add specific capabilities without deep hierarchies.

## Summary
- Inheritance enables code reuse through parent-child class relationships.
- `super()` delegates to the next class in the MRO, enabling cooperative multiple inheritance.
- The MRO follows C3 linearization: children before parents, left before right.
- Prefer composition over inheritance for "has-a" relationships.
- Use mixins for reusable, cross-cutting behavior like serialization or logging.

## Code Examples

**Mixin pattern**

```python
# Mixin classes for reusable behavior
class JSONMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)

class LoggingMixin:
    def log(self, message):
        print(f"[{self.__class__.__name__}] {message}")

class User(JSONMixin, LoggingMixin):
    def __init__(self, name):
        self.name = name

user = User("Alice")
print(user.to_json())  # '{"name": "Alice"}'
user.log("Created")     # [User] Created
```


## Resources

- [Inheritance](https://docs.python.org/3.14/tutorial/classes.html#inheritance) — Official Python 3.14 tutorial on inheritance, multiple inheritance, and the MRO

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*