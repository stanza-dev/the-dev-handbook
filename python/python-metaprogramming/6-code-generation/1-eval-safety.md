---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-exec-eval-safety"
---

# Dynamic Execution

*   `eval(expr)`: Evaluates a single expression and returns the value.
*   `exec(code)`: Executes a block of code (statements, class definitions, etc.) and returns `None`.

## The Dangers
**NEVER** run `exec` or `eval` on untrusted input. It allows arbitrary code execution (ACE).

## Restricted Scope
You can limit the scope by passing custom globals/locals dictionaries.

## Code Examples

**Sandboxed Eval**

```python
# Safe(r) evaluation
allowed_names = {"sum": sum, "range": range}
code = "sum(range(5))"

result = eval(code, {"__builtins__": {}}, allowed_names)
print(result) # 10

# If code was "import os; os.system('rm -rf /')", it would fail
# because __builtins__ is empty.
```


---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*