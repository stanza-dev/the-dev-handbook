---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-exec-eval-safety"
---

# Exec & Eval

## Introduction

Python ships with two powerful built-in functions -- `eval()` and `exec()` -- that let you execute code stored as strings at runtime. Understanding when and how to use them safely is essential for any developer building plugin systems, configuration-driven logic, or template engines, while misusing them is one of the fastest ways to introduce critical security vulnerabilities.

## Key Concepts

- **`eval()`**: Evaluates a single Python expression and returns its result.
- **`exec()`**: Executes an arbitrary block of Python statements and always returns `None`.
- **Namespace restriction**: The practice of passing custom `globals` and `locals` dictionaries to limit what names are accessible during dynamic execution.
- **Arbitrary code execution (ACE)**: A security vulnerability where an attacker can run any code on your system, often introduced by passing untrusted input to `eval()` or `exec()`.
- **`ast.literal_eval()`**: A safe alternative that only evaluates Python literal structures (strings, numbers, tuples, lists, dicts, booleans, and `None`).

## Real World Context

Imagine you are building a reporting tool that lets users type simple math formulas like `revenue - costs` into a configuration file. You could use `eval()` to compute the result dynamically. However, if you pipe raw user input straight into `eval()`, a malicious user could wipe your server. Knowing how to sandbox these functions -- and when to avoid them entirely -- is a day-one production skill.

## Deep Dive

### eval() -- Evaluate a Single Expression

The signature is `eval(expression, globals=None, locals=None)`. It parses and evaluates a single Python expression, returning its value. The following example shows basic arithmetic, variable lookup, and custom namespace usage:

```python
result = eval("2 + 3 * 4")
print(result)  # 14

# With variables
x = 10
print(eval("x ** 2"))  # 100

# With custom namespace
print(eval("a + b", {"a": 5, "b": 7}))  # 12
```

Notice that each call returns the computed value, making `eval()` ideal for dynamic formula evaluation. However, `eval()` only handles expressions -- things that produce a value. It cannot execute statements like `if`, `for`, `def`, `class`, or assignments. Attempting to do so raises a `SyntaxError`:

```python
eval("x = 5")  # SyntaxError: invalid syntax
```

This limitation is by design: `eval()` is intentionally narrow so you know exactly what it can do.

### exec() -- Execute Arbitrary Statements

When you need to run full Python code -- function definitions, class declarations, loops -- you reach for `exec(code, globals=None, locals=None)`. Unlike `eval()`, it always returns `None`; any results must be captured through the namespace dictionaries you pass in. Here is an example that defines both a function and a class dynamically:

```python
namespace = {}
exec("""
def greet(name):
    return f'Hello, {name}!'

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
""", namespace)

print(namespace["greet"]("Alice"))  # Hello, Alice!
p = namespace["Point"](1, 2)
print(p.x, p.y)  # 1 2
```

After execution, the `namespace` dictionary contains the `greet` function and the `Point` class, ready to be used like any other Python object.

### The Dangers of Untrusted Input

You should **never** run `exec()` or `eval()` on untrusted input. Doing so opens the door to arbitrary code execution, where an attacker can run any system command through Python. Consider this example of what a malicious user could supply:

```python
# A malicious user could input:
eval("__import__('os').system('rm -rf /')")
```

This single line could destroy an entire filesystem. The risk applies equally to `exec()`.

### Restricting the Scope

You can reduce the attack surface by passing custom `globals` and `locals` dictionaries that strip out dangerous builtins. The following example shows how to create a restricted evaluation environment:

```python
# Remove all builtins
result = eval("2 + 2", {"__builtins__": {}})

# Allow only specific names
safe_globals = {"__builtins__": {}, "sum": sum, "range": range}
result = eval("sum(range(10))", safe_globals)  # 45
```

While this is a useful first layer of defense, be aware that even with `{"__builtins__": {}}`, determined attackers can escape via introspection on built-in types. For truly safe evaluation of untrusted input, use `ast.literal_eval()` (which only parses literals) or a dedicated sandboxing library.

### eval() vs exec() at a Glance

The following table summarizes the key differences between the two functions:

| Feature | `eval()` | `exec()` |
|---------|----------|----------|
| Input | Single expression | Any code (statements, blocks) |
| Returns | The expression's value | `None` |
| Can define functions/classes | No | Yes |
| Can do assignments | No | Yes |

Choose `eval()` when you need a return value from a simple expression, and `exec()` when you need to execute full statements.

## Common Pitfalls

1. **Using `eval()` where `ast.literal_eval()` suffices.** If you only need to parse a Python literal (e.g., a list or dict from a config file), `ast.literal_eval()` is far safer because it refuses to execute arbitrary expressions.
2. **Assuming `{"__builtins__": {}}` is a complete sandbox.** Attackers can still reach dangerous functions through object introspection chains like `().__class__.__bases__[0].__subclasses__()`. Namespace restriction reduces risk but does not eliminate it.
3. **Forgetting that `exec()` returns `None`.** A common mistake is writing `result = exec("x = 5")` and expecting `result` to be `5`. You must retrieve values from the namespace dictionary instead.

## Best Practices

1. **Default to not using `eval()`/`exec()` at all.** In most cases, safer alternatives exist -- dictionaries for dispatch, `getattr()` for dynamic attribute access, or `ast.literal_eval()` for parsing literals.
2. **Always restrict the namespace** when dynamic execution is genuinely necessary. Pass explicit `globals` and `locals` dictionaries containing only the names the code needs.
3. **Log and audit all dynamically executed code** in production systems. If something goes wrong, you need a trail showing exactly what code was run and where it came from.

## Summary

- `eval()` evaluates a single expression and returns its value; `exec()` executes arbitrary statements and returns `None`.
- Both functions accept optional `globals` and `locals` dictionaries to restrict the available namespace.
- Running `eval()` or `exec()` on untrusted input is a critical security vulnerability -- always validate or sandbox input first.
- `ast.literal_eval()` is the safe choice for parsing Python literals from external sources.
- When you must use dynamic execution, restrict builtins, whitelist allowed names, and audit every invocation.

## Code Examples

**Sandboxed eval and dynamic function creation with exec**

```python
# Safe(r) evaluation with restricted builtins
allowed_names = {"sum": sum, "range": range, "len": len}
code = "sum(range(5))"

result = eval(code, {"__builtins__": {}}, allowed_names)
print(result)  # 10

# exec() to define a function dynamically
namespace = {}
exec("""
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)
""", namespace)

print(namespace["factorial"](5))  # 120
```


## Resources

- [undefined](https://docs.python.org/3.14/library/functions.html#compile) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*