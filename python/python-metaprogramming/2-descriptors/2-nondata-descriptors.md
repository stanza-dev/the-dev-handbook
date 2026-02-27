---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-data-vs-nondata-descriptors"
---

# Data vs Non-Data Descriptors

## Introduction

Not all descriptors are created equal. Python distinguishes between **data descriptors** and **non-data descriptors**, and this distinction directly controls attribute lookup priority. Understanding this difference is essential for predicting how Python resolves `obj.attr` and for building descriptors that behave correctly.

## Key Concepts

- **Data descriptor** -- Defines both `__get__` and at least one of `__set__` or `__delete__`. Data descriptors **take priority over the instance `__dict__`**.
- **Non-data descriptor** -- Defines only `__get__` (no `__set__`, no `__delete__`). Non-data descriptors **lose to the instance `__dict__`**.
- **Attribute lookup order** -- Python checks data descriptors first, then the instance dict, then non-data descriptors.

## Real World Context

- `property` is a **data descriptor** -- it always intercepts reads and writes, even if you set the same attribute name on the instance.
- Regular methods (plain functions) are **non-data descriptors** -- they define `__get__` to bind `self`, but have no `__set__`. That is why you can shadow a method with an instance attribute.
- `functools.cached_property` is deliberately a **non-data descriptor** -- on first access it computes the value and stores it in the instance dict. On subsequent accesses, the instance dict entry wins (because non-data descriptors lose to instance dict), so the descriptor is never called again.

## Deep Dive

### The Full Attribute Lookup Order

When you write `obj.attr`, Python follows this resolution chain:

```
1. type(obj).__mro__  ->  find 'attr' in the class chain
2. If found and it is a DATA descriptor  ->  call __get__  ->  DONE
3. If 'attr' is in obj.__dict__          ->  return it    ->  DONE
4. If found and it is a NON-DATA descriptor  ->  call __get__  ->  DONE
5. Raise AttributeError
```

This means data descriptors **cannot be overridden** by instance attributes, while non-data descriptors **can**.

### Demonstrating the Difference

```python
class DataDesc:
    """Data descriptor: has __get__ AND __set__."""
    def __get__(self, obj, objtype=None):
        print("DataDesc.__get__ called")
        return obj.__dict__.get('_val', 'default')

    def __set__(self, obj, value):
        print("DataDesc.__set__ called")
        obj.__dict__['_val'] = value


class NonDataDesc:
    """Non-data descriptor: has only __get__."""
    def __get__(self, obj, objtype=None):
        print("NonDataDesc.__get__ called")
        return 42


class MyClass:
    data_attr = DataDesc()
    nondata_attr = NonDataDesc()

obj = MyClass()

# --- Data descriptor wins over instance dict ---
obj.__dict__['data_attr'] = 'instance value'
print(obj.data_attr)
# DataDesc.__get__ called
# 'default'  <-- descriptor wins, instance dict ignored

# --- Non-data descriptor loses to instance dict ---
obj.__dict__['nondata_attr'] = 'instance value'
print(obj.nondata_attr)
# 'instance value'  <-- instance dict wins, descriptor skipped
```

### Why `property` Always Wins

```python
class Example:
    @property
    def x(self):
        return 'from property'

e = Example()
e.__dict__['x'] = 'from dict'
print(e.x)  # 'from property' -- property is a data descriptor
```

`property` defines both `__get__` and `__set__` (and `__delete__`), making it a data descriptor. The instance dict entry `'x'` is completely ignored.

### How `cached_property` Exploits Non-Data Behavior

```python
from functools import cached_property

class Config:
    @cached_property
    def db_url(self):
        print("Computing...")
        return "postgresql://localhost/mydb"

c = Config()
print(c.db_url)  # Computing... -> postgresql://localhost/mydb
print(c.db_url)  # postgresql://localhost/mydb (no recomputation)
```

`cached_property` only defines `__get__`. On first access, it stores the result in `obj.__dict__['db_url']`. On second access, the instance dict entry wins because non-data descriptors have lower priority.

### Making a Read-Only Descriptor

To make a descriptor read-only, define `__set__` but raise an error:

```python
class ReadOnly:
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(f'_ro_{self.name}')

    def __set__(self, obj, value):
        if f'_ro_{self.name}' in obj.__dict__:
            raise AttributeError(f"{self.name} is read-only")
        obj.__dict__[f'_ro_{self.name}'] = value

class Settings:
    api_key = ReadOnly()

    def __init__(self, api_key):
        self.api_key = api_key  # First set OK

s = Settings("secret-123")
print(s.api_key)       # secret-123
# s.api_key = "new"   # AttributeError: api_key is read-only
```

Because `ReadOnly` defines `__set__`, it is a data descriptor -- so the instance dict can never shadow it.

## Common Pitfalls

- **Assuming all descriptors have the same priority.** A non-data descriptor can be silently overridden by an instance attribute, which can cause confusing bugs if you expected the descriptor to always be called.
- **Adding `__set__` when you want caching behavior.** If your descriptor defines `__set__`, it becomes a data descriptor and the instance dict can never override it -- breaking the caching pattern used by `cached_property`.
- **Forgetting that functions are descriptors.** Regular functions define `__get__` (to bind `self`), which is why `obj.method` returns a bound method. They are non-data descriptors, so you can shadow a method with an instance attribute.

## Best Practices

- Use **data descriptors** when you need to intercept every read and write (validation, logging, computed properties).
- Use **non-data descriptors** when you want a one-time computation that caches in the instance dict.
- If you want a read-only attribute, make it a data descriptor by defining `__set__` that raises `AttributeError`.
- Remember the lookup order: data descriptor > instance dict > non-data descriptor.

## Summary

Data descriptors (defining `__get__` + `__set__` or `__delete__`) always take priority over the instance `__dict__`. Non-data descriptors (defining only `__get__`) lose to the instance dict, which is what makes patterns like `cached_property` possible. The attribute lookup order -- data descriptor, then instance dict, then non-data descriptor -- is the key mental model for understanding Python's attribute resolution.

## Code Examples

**Side-by-side comparison showing data descriptors win over instance dict but non-data descriptors do not.**

```python
class DataDesc:
    """Data descriptor: defines __get__ and __set__."""
    def __get__(self, obj, objtype=None):
        return obj.__dict__.get('_data', 'DATA default')
    def __set__(self, obj, value):
        obj.__dict__['_data'] = value

class NonDataDesc:
    """Non-data descriptor: defines only __get__."""
    def __get__(self, obj, objtype=None):
        return 'NON-DATA default'

class Demo:
    a = DataDesc()
    b = NonDataDesc()

d = Demo()

# Inject values directly into instance dict
d.__dict__['a'] = 'instance a'
d.__dict__['b'] = 'instance b'

# Data descriptor wins over instance dict
print(d.a)  # 'DATA default' (descriptor __get__ called)

# Non-data descriptor loses to instance dict
print(d.b)  # 'instance b' (instance dict wins)
```

**functools.cached_property leverages non-data descriptor behavior for automatic caching.**

```python
from functools import cached_property

class ExpensiveResource:
    @cached_property
    def connection(self):
        """Simulates an expensive connection."""
        print("Establishing connection...")
        return {"host": "db.example.com", "port": 5432}

r = ExpensiveResource()
print(r.connection)  # Establishing connection... -> {...}
print(r.connection)  # {...} (cached, descriptor not called)

# You can invalidate the cache by deleting the instance dict entry
del r.__dict__['connection']
print(r.connection)  # Establishing connection... -> {...}
```


## Resources

- [Descriptor HowTo -- Data and Non-Data Descriptors](https://docs.python.org/3.14/howto/descriptor.html#data-and-non-data-descriptors) — undefined
- [functools.cached_property](https://docs.python.org/3.14/library/functools.html#functools.cached_property) — undefined
- [Data Model -- Invoking Descriptors](https://docs.python.org/3.14/reference/datamodel.html#invoking-descriptors) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*