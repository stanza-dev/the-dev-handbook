---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-practical-descriptors"
---

# Building Practical Descriptors

## Introduction

Now that you understand the descriptor protocol and the data vs non-data distinction, it is time to build real-world descriptors. This lesson covers validated fields, cached properties, type-checked attributes, and ORM-like field definitions -- the same patterns used by Django, SQLAlchemy, and attrs.

## Key Concepts

- **Validated fields** -- Descriptors that enforce constraints (range, format, type) on every assignment.
- **Cached properties** -- Non-data descriptors that compute once and store the result on the instance.
- **Type-checked attributes** -- Descriptors that reject values of the wrong type at assignment time.
- **ORM-like fields** -- Descriptors combined with `__init_subclass__` to build declarative APIs where class attributes define a schema.

## Real World Context

- Django model fields (`CharField(max_length=100)`) are descriptors that validate data before it reaches the database.
- SQLAlchemy's `mapped_column()` uses descriptors to map Python attributes to database columns.
- The `attrs` and `pydantic` libraries use descriptor-like patterns (via `__set_attr__` or model validators) for declarative data classes.
- Python's own `property` is a descriptor that provides getter/setter/deleter with a clean syntax.

## Deep Dive

### Pattern 1: Validated Field Descriptor

A generic descriptor that accepts a validation function:

```python
class ValidatedField:
    """Descriptor with pluggable validation."""

    def __init__(self, *validators):
        self.validators = validators

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        for validator in self.validators:
            validator(self.name, value)  # Raises on failure
        obj.__dict__[self.name] = value


# Reusable validators
def positive(name, value):
    if value <= 0:
        raise ValueError(f"{name} must be positive, got {value}")

def max_length(limit):
    def check(name, value):
        if len(value) > limit:
            raise ValueError(f"{name} exceeds max length {limit}")
    return check

def one_of(*choices):
    def check(name, value):
        if value not in choices:
            raise ValueError(f"{name} must be one of {choices}")
    return check


class Product:
    name = ValidatedField(max_length(50))
    price = ValidatedField(positive)
    category = ValidatedField(one_of("electronics", "books", "clothing"))

    def __init__(self, name, price, category):
        self.name = name
        self.price = price
        self.category = category

p = Product("Laptop", 999.99, "electronics")  # OK
# Product("Laptop", -10, "electronics")       # ValueError
# Product("Laptop", 999, "food")              # ValueError
```

### Pattern 2: Cached Property (Manual Implementation)

This mirrors `functools.cached_property` to show how it works under the hood:

```python
class CachedProperty:
    """Non-data descriptor that caches the result on the instance."""

    def __init__(self, func):
        self.func = func
        self.name = func.__name__
        self.__doc__ = func.__doc__

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        # Compute and store directly in instance dict
        value = self.func(obj)
        obj.__dict__[self.name] = value
        return value


class Report:
    def __init__(self, data):
        self.data = data

    @CachedProperty
    def summary(self):
        """Expensive aggregation."""
        print("Computing summary...")
        return {
            "count": len(self.data),
            "total": sum(self.data),
            "mean": sum(self.data) / len(self.data),
        }

r = Report([10, 20, 30, 40, 50])
print(r.summary)  # Computing summary...
print(r.summary)  # cached, no recomputation
```

Because `CachedProperty` is a non-data descriptor (no `__set__`), the `obj.__dict__[self.name] = value` line stores the result where Python looks **before** falling back to the descriptor. The descriptor is never called again.

### Pattern 3: Type-Checked Attributes

```python
class Typed:
    """Data descriptor that enforces a specific type."""

    def __init__(self, expected_type, *, default=None):
        self.expected_type = expected_type
        self.default = default

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name, self.default)

    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"{self.name}: expected {self.expected_type.__name__}, "
                f"got {type(value).__name__}"
            )
        obj.__dict__[self.name] = value


class Config:
    host = Typed(str, default="localhost")
    port = Typed(int, default=8080)
    debug = Typed(bool, default=False)

    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            setattr(self, key, value)

c = Config(host="db.example.com", port=5432, debug=True)
print(c.host)   # db.example.com
print(c.port)   # 5432
# Config(port="not-a-number")  # TypeError
```

### Pattern 4: ORM-Like Field Definitions

Combine descriptors with `__init_subclass__` to build a mini declarative ORM:

```python
class Field:
    """Descriptor acting as an ORM column definition."""

    def __init__(self, field_type, required=True, default=None):
        self.field_type = field_type
        self.required = required
        self.default = default

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name, self.default)

    def __set__(self, obj, value):
        if value is not None and not isinstance(value, self.field_type):
            raise TypeError(
                f"{self.name}: expected {self.field_type.__name__}"
            )
        obj.__dict__[self.name] = value


class Model:
    """Base class that auto-generates __init__ from Field descriptors."""

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        cls._fields = {
            name: attr for name, attr in vars(cls).items()
            if isinstance(attr, Field)
        }

    def __init__(self, **kwargs):
        for name, field in self.__class__._fields.items():
            if name in kwargs:
                setattr(self, name, kwargs[name])
            elif field.required and field.default is None:
                raise ValueError(f"Missing required field: {name}")
            else:
                setattr(self, name, field.default)

    def __repr__(self):
        attrs = ', '.join(
            f"{n}={getattr(self, n)!r}" for n in self.__class__._fields
        )
        return f"{self.__class__.__name__}({attrs})"


class User(Model):
    name = Field(str)
    email = Field(str)
    age = Field(int, required=False, default=0)

u = User(name="Alice", email="alice@example.com")
print(u)       # User(name='Alice', email='alice@example.com', age=0)
print(u.age)   # 0
u.age = 25
print(u.age)   # 25
# User(name="Alice")  # ValueError: Missing required field: email
```

## Common Pitfalls

- **Overcomplicating simple use cases.** If you only need a getter and setter for one attribute, use `@property`. Descriptors shine when the same logic must apply to many attributes across many classes.
- **Forgetting `__set_name__`.** Without it, every field instantiation requires a redundant name string argument.
- **Not handling `None` or default values.** Make sure your validators handle `None` if the field is optional.
- **Mutable defaults.** If your descriptor has a mutable default (like a list), each instance will share the same object. Use a factory pattern instead: `default_factory=list`.

## Best Practices

- Build small, composable validators and combine them with a generic `ValidatedField` descriptor.
- Use non-data descriptors for caching patterns and data descriptors for validation patterns.
- Combine descriptors with `__init_subclass__` for declarative APIs that feel Pythonic.
- Write thorough tests for edge cases: `None` values, wrong types, missing required fields, and class-level access (`MyClass.attr`).

## Summary

Practical descriptors follow a few recurring patterns: validated fields (pluggable validators on a data descriptor), cached properties (non-data descriptors that store results in the instance dict), type-checked attributes (data descriptors that enforce `isinstance` checks), and ORM-like fields (descriptors combined with `__init_subclass__` for declarative class definitions). These patterns are the building blocks of Python frameworks like Django, SQLAlchemy, attrs, and pydantic.

## Code Examples

**ValidatedField descriptor with composable validators for building reusable, declarative validation.**

```python
class ValidatedField:
    """Generic descriptor with pluggable validators."""
    def __init__(self, *validators):
        self.validators = validators

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        for validator in self.validators:
            validator(self.name, value)
        obj.__dict__[self.name] = value


def between(low, high):
    def check(name, value):
        if not (low <= value <= high):
            raise ValueError(f"{name} must be between {low} and {high}")
    return check

def non_empty(name, value):
    if not value:
        raise ValueError(f"{name} must not be empty")


class StudentGrade:
    name = ValidatedField(non_empty)
    score = ValidatedField(between(0, 100))

    def __init__(self, name, score):
        self.name = name
        self.score = score

g = StudentGrade("Alice", 95)   # OK
print(g.name, g.score)          # Alice 95
# StudentGrade("", 95)          # ValueError: name must not be empty
# StudentGrade("Alice", 150)    # ValueError: score must be between 0 and 100
```

**Mini ORM using descriptors and __init_subclass__ for declarative model definitions.**

```python
class Field:
    """ORM-like field descriptor."""
    def __init__(self, field_type, required=True, default=None):
        self.field_type = field_type
        self.required = required
        self.default = default

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name, self.default)

    def __set__(self, obj, value):
        if value is not None and not isinstance(value, self.field_type):
            raise TypeError(f"{self.name}: expected {self.field_type.__name__}")
        obj.__dict__[self.name] = value


class Model:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        cls._fields = {
            k: v for k, v in vars(cls).items() if isinstance(v, Field)
        }

    def __init__(self, **kwargs):
        for name, field in self._fields.items():
            if name in kwargs:
                setattr(self, name, kwargs[name])
            elif field.required and field.default is None:
                raise ValueError(f"Missing required field: {name}")

    def __repr__(self):
        parts = ', '.join(f"{n}={getattr(self, n)!r}" for n in self._fields)
        return f"{type(self).__name__}({parts})"


class BlogPost(Model):
    title = Field(str)
    body = Field(str)
    views = Field(int, required=False, default=0)

post = BlogPost(title="Descriptors", body="They are powerful")
print(post)  # BlogPost(title='Descriptors', body='They are powerful', views=0)
```


## Resources

- [Descriptor HowTo Guide](https://docs.python.org/3.14/howto/descriptor.html) — undefined
- [Built-in Functions -- property](https://docs.python.org/3.14/library/functions.html#property) — undefined
- [functools.cached_property](https://docs.python.org/3.14/library/functools.html#functools.cached_property) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*