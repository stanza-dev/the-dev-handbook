---
source_course: "python"
source_lesson: "python-class-basics"
---

# Classes in Python

Classes are blueprints for creating objects with state (attributes) and behavior (methods).

## Defining a Class

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

## Creating Objects

```python
my_dog = Dog("Buddy", 3)
print(my_dog.name)    # "Buddy"
print(my_dog.bark())  # "Buddy says woof!"
print(Dog.species)    # "Canis familiaris"
```

## Instance vs Class Attributes

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

## Special Methods (Dunder Methods)

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*