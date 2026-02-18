---
source_course: "python-performance"
source_lesson: "python-performance-weak-references"
---

# Weak References

Weak references don't increment the reference count.

## Basic Usage

```python
import weakref

class BigObject:
    def __init__(self, data):
        self.data = data

obj = BigObject("important")
weak = weakref.ref(obj)

print(weak())  # <BigObject object>
del obj
print(weak())  # None - object was collected
```

## Weak Callbacks

```python
def callback(ref):
    print("Object was deleted!")

obj = BigObject("data")
weak = weakref.ref(obj, callback)
del obj  # Prints: "Object was deleted!"
```

## WeakValueDictionary

Perfect for caches:

```python
import weakref

# Cache that doesn't prevent garbage collection
cache = weakref.WeakValueDictionary()

def get_user(user_id):
    if user_id in cache:
        return cache[user_id]
    
    user = fetch_from_db(user_id)
    cache[user_id] = user
    return user

# When no other references exist, users are freed
```

## WeakSet

```python
import weakref

# Track instances without preventing deletion
class Trackable:
    _instances = weakref.WeakSet()
    
    def __init__(self):
        Trackable._instances.add(self)
    
    @classmethod
    def get_all(cls):
        return list(cls._instances)
```

## functools.lru_cache

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive(n):
    return compute(n)

# Check cache stats
print(expensive.cache_info())

# Clear cache
expensive.cache_clear()
```

## Code Examples

**Weak method references**

```python
import weakref

# Weak reference to method (tricky!)
class MyClass:
    def method(self):
        return "Hello"

obj = MyClass()

# This doesn't work - bound methods are recreated each time
# weak = weakref.ref(obj.method)  # Would be dead immediately

# Use WeakMethod instead
from weakref import WeakMethod

weak_method = WeakMethod(obj.method)
print(weak_method()())  # "Hello"

del obj
print(weak_method())  # None
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*