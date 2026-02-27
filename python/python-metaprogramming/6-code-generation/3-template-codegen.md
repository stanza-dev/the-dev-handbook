---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-template-codegen"
---

# Template-Based Code Generation

## Introduction

Sometimes the best way to eliminate boilerplate is to have Python write Python for you. Template-based code generation combines string formatting with `compile()` and `exec()` to create classes, functions, and validators programmatically at runtime. This is the exact technique that powers Python's own `dataclasses` and `namedtuple` implementations under the hood.

## Key Concepts

- **Template-based code generation**: Building Python source code as a string using string formatting, then compiling and executing it to produce real classes or functions.
- **Schema-driven code**: Generating code from external data like database schemas, JSON schemas, or API specifications, so that the structure of the code is determined by data rather than written by hand.
- **`__slots__`**: A class-level declaration that restricts instances to a fixed set of attributes, saving memory and preventing typos in attribute names.
- **DTO / struct pattern**: A lightweight data-carrying class with `__init__`, `__repr__`, and `__eq__` methods -- exactly what code generation excels at producing.

## Real World Context

You are building an ORM that reads table definitions from a database and needs to generate a Python class for each table, complete with typed `__init__`, `__repr__`, and validation methods. Writing these classes by hand for hundreds of tables would be tedious and error-prone. Instead, you write a code generation function once, and it produces correct, performant classes for every table automatically. This is not a hypothetical scenario -- it is how SQLAlchemy, Pydantic, attrs, and dataclasses actually work.

## Deep Dive

### Why Generate Code?

There are three primary reasons to reach for code generation:

- **Schema-driven models**: Generate classes from database schemas, JSON schemas, or API specs.
- **Boilerplate elimination**: Create repetitive methods (getters, setters, validators) automatically.
- **Performance**: Generated code runs at full speed, unlike approaches that intercept every attribute access at runtime.

This is exactly how Python's `dataclasses` and `namedtuple` work internally.

### Generating a Class from a Schema

The core pattern is straightforward: build a source string using f-strings, compile it, execute it into a namespace, and extract the resulting class. The following `make_struct` function generates a class with `__init__`, `__repr__`, and `__eq__` from a list of field names:

```python
def make_struct(name, fields):
    """Generate a class with __init__, __repr__, and __eq__ from a field list."""
    
    # Build __init__ parameters and body
    params = ", ".join(fields)
    init_body = "\n".join(f"        self.{f} = {f}" for f in fields)
    
    # Build __repr__
    repr_parts = ", ".join(f"{f}={{self.{f}!r}}" for f in fields)
    
    # Build __eq__
    eq_checks = " and ".join(f"self.{f} == other.{f}" for f in fields)
    
    source = f"""
class {name}:
    __slots__ = {tuple(fields)}
    
    def __init__(self, {params}):
{init_body}
    
    def __repr__(self):
        return f"{name}({repr_parts})"
    
    def __eq__(self, other):
        if not isinstance(other, {name}):
            return NotImplemented
        return {eq_checks}
"""
    
    namespace = {}
    exec(compile(source, f"<generated:{name}>", "exec"), namespace)
    return namespace[name]

# Usage
Point = make_struct("Point", ["x", "y"])
p = Point(3, 4)
print(p)            # Point(x=3, y=4)
print(p == Point(3, 4))  # True
```

The generated class uses `__slots__` for memory efficiency and includes `NotImplemented` handling in `__eq__` so that comparisons with unrelated types behave correctly.

### Using compile() for Better Tracebacks

The `filename` argument in `compile()` appears in tracebacks when errors occur. Choosing descriptive filenames makes debugging generated code dramatically easier:

```python
# Bad: errors show "<string>"
exec(compile(source, "<string>", "exec"))

# Good: errors show "<generated:Point>"
exec(compile(source, "<generated:Point>", "exec"))

# Even better: errors show the schema file that triggered generation
exec(compile(source, f"schema://{schema_path}:Point", "exec"))
```

When a generated method raises an exception, the traceback will include whichever filename you provided, so invest a few characters in making it informative.

### How dataclasses Does It

Python's `dataclasses` module generates `__init__`, `__repr__`, `__eq__`, `__hash__`, and comparison methods using exactly this pattern. The following is a simplified version of the internal `_create_fn` function from CPython's `dataclasses.py`:

```python
# Simplified from CPython's dataclasses.py
def _create_fn(name, args, body, *, globals=None, locals=None):
    txt = f"def {name}({args}):\n"
    for line in body:
        txt += f"  {line}\n"
    
    local_vars = {}
    exec(compile(txt, "<dataclass>", "exec"), globals, local_vars)
    return local_vars[name]
```

This is the same compile-and-exec pattern we used in `make_struct`. The `dataclasses` module just applies it to each individual method rather than an entire class.

### Generating Validators from a Schema

Code generation is not limited to data classes. You can also generate validation functions from declarative schema dictionaries. The following function builds a validator that checks types and minimum values based on a schema:

```python
def make_validator(schema: dict) -> callable:
    """Generate a validation function from a schema dict."""
    checks = []
    for field, rules in schema.items():
        if "type" in rules:
            type_name = rules["type"].__name__
            checks.append(
                f"    if not isinstance(data.get('{field}'), {type_name}):"
                f"\n        errors.append('{field} must be {type_name}')"
            )
        if "min" in rules:
            checks.append(
                f"    if data.get('{field}', 0) < {rules['min']}:"
                f"\n        errors.append('{field} must be >= {rules['min']}')"
            )
    
    source = "def validate(data):\n    errors = []\n"
    source += "\n".join(checks)
    source += "\n    return errors\n"
    
    namespace = {"int": int, "str": str, "float": float}
    exec(compile(source, "<validator>", "exec"), namespace)
    return namespace["validate"]

# Usage
schema = {
    "name": {"type": str},
    "age":  {"type": int, "min": 0},
}
validate = make_validator(schema)
print(validate({"name": "Alice", "age": -1}))  # ['age must be >= 0']
```

The generated `validate` function runs at full speed because it is real compiled Python code -- there is no dictionary lookup or conditional dispatch at validation time.

### When Code Generation Is Appropriate

Code generation is a powerful tool, but it is not always the right one. Here is a quick decision framework:

**Good use cases:**
- Eliminating large amounts of boilerplate (dataclasses, ORMs)
- Schema-driven code where the structure is only known at runtime
- Performance-critical paths where dynamic dispatch is too slow

**Code generation is overengineering when:**
- A simple class hierarchy or dictionary would suffice
- The generated code is only used once
- The template logic is more complex than writing the code directly
- You can achieve the same result with descriptors or `__init_subclass__`

## Common Pitfalls

1. **Generating code that is impossible to debug.** If your generated source strings are built through many layers of concatenation, the final code becomes unreadable. Always store or log the complete generated source so you can inspect it when something goes wrong.
2. **Not sanitizing field names.** If field names come from external data (like a database schema), they could contain Python keywords or special characters that break the generated code. Always validate that field names are valid Python identifiers using `str.isidentifier()`.
3. **Overusing code generation when simpler tools exist.** If your use case can be solved with `__init_subclass__`, descriptors, or a simple factory function, those approaches are easier to understand, test, and debug than string-based code generation.

## Best Practices

1. **Use `compile()` with descriptive filenames** so that tracebacks from generated code are traceable back to the schema or configuration that produced them.
2. **Generate the source string cleanly and log it.** Build the source in a readable way (using dedented multi-line f-strings), and consider logging or caching the generated source for debugging purposes.
3. **Write tests that verify generated code.** Since generated classes and functions are not visible in your source tree, they need explicit test coverage to catch regressions when you change the generation logic.

## Summary

- Template-based code generation builds Python source as strings, then compiles and executes them to produce real classes and functions.
- This is the pattern used internally by `dataclasses`, `namedtuple`, and many ORMs to eliminate boilerplate.
- The `filename` argument in `compile()` should always be descriptive to ensure useful tracebacks from generated code.
- Code generation is ideal for schema-driven, repetitive, or performance-sensitive scenarios, but should be avoided when simpler alternatives like descriptors or `__init_subclass__` would suffice.
- Always validate external input used in templates (e.g., field names) and log the generated source for debuggability.

## Code Examples

**Generate struct-like classes from field lists, similar to dataclasses internals**

```python
def make_struct(name, fields):
    """Generate a struct-like class from a field list."""
    params = ", ".join(fields)
    init_body = "\n".join(f"        self.{f} = {f}" for f in fields)
    repr_parts = ", ".join(f"{f}={{self.{f}!r}}" for f in fields)
    eq_checks = " and ".join(f"self.{f} == other.{f}" for f in fields)

    source = f"""
class {name}:
    __slots__ = {tuple(fields)}

    def __init__(self, {params}):
{init_body}

    def __repr__(self):
        return f"{name}({repr_parts})"

    def __eq__(self, other):
        if not isinstance(other, {name}):
            return NotImplemented
        return {eq_checks}
"""
    namespace = {}
    exec(compile(source, f"<generated:{name}>", "exec"), namespace)
    return namespace[name]

# Create classes dynamically
Point = make_struct("Point", ["x", "y"])
Color = make_struct("Color", ["r", "g", "b"])

p = Point(3, 4)
c = Color(255, 128, 0)
print(p)  # Point(x=3, y=4)
print(c)  # Color(r=255, g=128, b=0)
print(p == Point(3, 4))  # True
```


## Resources

- [undefined](https://docs.python.org/3.14/library/functions.html#compile) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*