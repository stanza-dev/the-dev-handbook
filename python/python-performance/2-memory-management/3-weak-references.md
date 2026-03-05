---
source_course: "python-performance"
source_lesson: "python-performance-weak-references"
---

# Weak References & Caches

## Introduction

Weak references let you refer to an object without preventing it from being garbage collected. This is essential for building caches, observer patterns, and object registries that do not cause memory leaks. Python's `weakref` module provides several tools for this purpose, and `functools` provides high-level caching decorators.

## Key Concepts

- **Weak Reference (`weakref.ref`)**: A reference to an object that does not increment its reference count. When the object is collected, the weak reference returns `None`.
- **WeakValueDictionary**: A dictionary where the values are weak references. Entries are automatically removed when the value objects are garbage collected.
- **WeakKeyDictionary**: A dictionary where the keys are weak references. Useful for attaching metadata to objects without preventing their collection.
- **WeakSet**: A set that holds weak references to its elements. Elements are automatically removed when collected.
- **`functools.lru_cache`**: A decorator that caches function return values with a bounded Least Recently Used eviction policy.

## Real World Context

In a web application, you might cache database query results in memory to avoid repeated queries. If you use a regular dictionary, those cached objects stay in memory forever. A `WeakValueDictionary` lets the cache entries be freed automatically when no other part of the application needs them, preventing memory leaks without manual cache invalidation.

## Deep Dive

### Basic Weak References

A weak reference does not keep an object alive:

```python
import weakref

class ExpensiveResource:
    def __init__(self, name):
        self.name = name
        self.data = bytearray(1_000_000)  # 1 MB of data

# Create the object and a weak reference to it
resource = ExpensiveResource("dataset")
weak_ref = weakref.ref(resource)

# The weak reference works while the object exists
print(weak_ref())          # <ExpensiveResource object>
print(weak_ref().name)     # "dataset"

# Delete the strong reference
del resource

# The object is now gone — weak reference returns None
print(weak_ref())          # None
```

### Weak Reference Callbacks

You can register a callback that fires when the referenced object is collected:

```python
import weakref

def on_collected(ref):
    print(f"Object collected! Weak ref: {ref}")

class Sensor:
    def __init__(self, sensor_id):
        self.sensor_id = sensor_id

sensor = Sensor(42)
weak = weakref.ref(sensor, on_collected)

del sensor  # Prints: "Object collected! Weak ref: ..."
```

### WeakValueDictionary for Caching

`WeakValueDictionary` is perfect for caches that should not prevent objects from being freed:

```python
import weakref

class UserProfile:
    def __init__(self, user_id, name):
        self.user_id = user_id
        self.name = name

# Cache using weak values
profile_cache = weakref.WeakValueDictionary()

def get_profile(user_id):
    """Return cached profile, or fetch and cache it."""
    if user_id in profile_cache:
        return profile_cache[user_id]

    # Simulate database fetch
    profile = UserProfile(user_id, f"User {user_id}")
    profile_cache[user_id] = profile
    return profile

# Use the profile
profile = get_profile(123)
print(f"Cache size: {len(profile_cache)}")  # 1

# When no other code holds a reference,
# the entry is automatically removed
del profile
# profile_cache[123] is now gone
```

### WeakSet for Instance Tracking

Track all instances of a class without preventing their collection:

```python
import weakref

class DatabaseConnection:
    _active_connections = weakref.WeakSet()

    def __init__(self, host):
        self.host = host
        DatabaseConnection._active_connections.add(self)

    @classmethod
    def active_count(cls):
        return len(cls._active_connections)

conn1 = DatabaseConnection("db1.example.com")
conn2 = DatabaseConnection("db2.example.com")
print(DatabaseConnection.active_count())  # 2

del conn1
print(DatabaseConnection.active_count())  # 1
```

### WeakMethod for Bound Methods

Bound methods are recreated on each access, so regular `weakref.ref` does not work with them. Use `weakref.WeakMethod` instead:

```python
from weakref import WeakMethod

class EventHandler:
    def on_event(self, data):
        print(f"Handling: {data}")

handler = EventHandler()
weak_callback = WeakMethod(handler.on_event)

# Call through the weak method
callback = weak_callback()
if callback is not None:
    callback("test data")  # Prints: "Handling: test data"

del handler
print(weak_callback())  # None — handler was collected
```

### functools Caching Decorators

For function-level caching, Python provides high-level decorators:

```python
from functools import lru_cache, cache

@lru_cache(maxsize=256)
def fibonacci(n):
    """Bounded cache with LRU eviction."""
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(100))
print(fibonacci.cache_info())
# CacheInfo(hits=98, misses=101, maxsize=256, currsize=101)

@cache  # Unbounded cache (Python 3.9+)
def factorial(n):
    return n * factorial(n - 1) if n else 1
```

## Common Pitfalls

- **Using `weakref.ref` with bound methods**: Bound methods are temporary objects created on attribute access. A `weakref.ref` to a bound method becomes dead immediately. Use `weakref.WeakMethod` instead.
- **Not checking if a weak reference is still alive**: Always check `weak_ref() is not None` before using the result. The referenced object could have been collected between when you obtained the reference and when you dereference it.
- **Using `@cache` (unbounded) for functions with many unique arguments**: `@cache` never evicts entries. If the function is called with millions of unique arguments, the cache will consume unbounded memory. Use `@lru_cache(maxsize=N)` for bounded caching.

## Best Practices

- **Use `WeakValueDictionary` for object caches**: This prevents the cache itself from keeping objects alive, avoiding memory leaks in long-running applications.
- **Prefer `@lru_cache` over manual caching**: The built-in decorator handles thread safety, eviction, and statistics. Only implement custom caches when you need time-based expiration or custom eviction.
- **Set explicit `maxsize` for `@lru_cache`**: Always specify a `maxsize` based on your expected usage to prevent unbounded memory growth.

## Summary

- Weak references (`weakref.ref`) allow you to reference objects without preventing their garbage collection
- `WeakValueDictionary` and `WeakSet` automatically remove entries when referenced objects are collected, making them ideal for caches and registries
- `WeakMethod` is required for weak references to bound methods, since regular `weakref.ref` does not work with them
- `functools.lru_cache` provides bounded caching with LRU eviction, while `@cache` provides unbounded caching
- Always check if a weak reference is still alive before using it (`ref() is not None`)

## Code Examples

**Using WeakValueDictionary as a texture cache that automatically evicts entries when no other code holds a reference to the cached objects**

```python
import weakref

class CachedTexture:
    """Simulates a large texture that should be cached weakly."""
    def __init__(self, name, width, height):
        self.name = name
        self.pixels = bytearray(width * height * 4)  # RGBA

# WeakValueDictionary: entries vanish when textures are collected
texture_cache = weakref.WeakValueDictionary()

def load_texture(name, width=512, height=512):
    if name in texture_cache:
        print(f"Cache hit: {name}")
        return texture_cache[name]
    print(f"Loading: {name}")
    tex = CachedTexture(name, width, height)
    texture_cache[name] = tex
    return tex

# First load: creates the texture
sky = load_texture("sky")     # "Loading: sky"
sky2 = load_texture("sky")    # "Cache hit: sky"

print(f"Cache entries: {len(texture_cache)}")  # 1

# Delete all strong references — cache entry auto-removed
del sky, sky2
print(f"Cache entries: {len(texture_cache)}")  # 0
```


## Resources

- [weakref — Weak references](https://docs.python.org/3.14/library/weakref.html) — Official documentation for the weakref module including ref, WeakValueDictionary, WeakSet, and WeakMethod
- [functools.lru_cache](https://docs.python.org/3.14/library/functools.html#functools.lru_cache) — Official documentation for the LRU cache decorator and related caching utilities

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*