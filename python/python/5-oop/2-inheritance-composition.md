---
source_course: "python"
source_lesson: "python-inheritance-composition"
---

# Inheritance

Inheritance allows classes to inherit attributes and methods from parent classes.

## Basic Inheritance

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

## Using super()

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

## Multiple Inheritance & MRO

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

## Composition Over Inheritance

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*