---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-inspect-module"
---

# Runtime Introspection

The `inspect` module provides functions to get information about live objects, including modules, classes, methods, functions, tracebacks, frame objects, and code objects.

```python
import inspect

def my_func(a, b=10):
    pass

sig = inspect.signature(my_func)
print(sig.parameters['b'].default) # 10
```

## Accessing Stack Frames
You can look up the call stack to see who called the current function (useful for debugging or advanced frameworks).

## Code Examples

**Inspecting Stack Frames**

```python
import inspect

def who_called_me():
    frame = inspect.currentframe()
    caller = frame.f_back
    print(f"Called by {caller.f_code.co_name}")

def main():
    who_called_me()

main() # Called by main
```


---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*