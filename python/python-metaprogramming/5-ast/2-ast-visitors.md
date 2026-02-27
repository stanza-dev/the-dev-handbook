---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-ast-visitors"
---

# AST Node Types & Visitors

## Introduction

Parsing code into an AST is only useful if you know what the nodes in the tree represent and how to traverse them systematically. The `ast` module defines dozens of node types -- one for each syntactic construct in Python -- and provides the `NodeVisitor` pattern for walking the tree in a structured, extensible way. Mastering node types and visitors is what turns raw AST knowledge into practical tools like code analyzers, import mappers, and complexity checkers.

## Key Concepts

- **`ast.dump()`**: Produces a human-readable string representation of an AST, showing every node type and its fields.
- **Node types**: Each Python construct has a corresponding AST class, such as `ast.FunctionDef`, `ast.Call`, `ast.Name`, and `ast.Constant`.
- **`ast.NodeVisitor`**: A base class that walks the tree and dispatches to `visit_<NodeType>` methods you define for each node type you care about.
- **`generic_visit()`**: The method that continues traversal into child nodes; forgetting to call it means nested nodes are silently skipped.

## Real World Context

Imagine you are building a dependency analysis tool that maps which modules your project imports, or a code reviewer that flags functions with too many arguments. Both tasks require iterating over an AST and collecting information from specific node types. The `NodeVisitor` pattern gives you a clean, object-oriented way to do this -- define a method per node type, collect your data, and let the framework handle the traversal mechanics.

## Deep Dive

Before you can work with specific node types, you need to see what the tree actually looks like. The `ast.dump()` function produces a string representation of the entire tree structure, which is invaluable for understanding how Python represents your code internally.

```python
import ast

tree = ast.parse("x = func(1, y)")
print(ast.dump(tree, indent=2))
```

This output shows you the exact nesting of nodes. The simplified output looks like this:

```
Module(
  body=[
    Assign(
      targets=[Name(id='x', ctx=Store())],
      value=Call(
        func=Name(id='func', ctx=Load()),
        args=[Constant(value=1), Name(id='y', ctx=Load())],
        keywords=[]))])
```

Notice how every piece of the expression maps to a specific node: the variable `x` is a `Name` node with `ctx=Store()`, the function call is a `Call` node, and the literal `1` is a `Constant` node. Here is a reference table of the most common node types you will encounter.

| Node | Represents | Example |
|------|------------|---------|
| `ast.FunctionDef` | Function definition | `def foo(): ...` |
| `ast.AsyncFunctionDef` | Async function | `async def foo(): ...` |
| `ast.ClassDef` | Class definition | `class Foo: ...` |
| `ast.Assign` | Assignment | `x = 1` |
| `ast.Call` | Function call | `foo(a, b)` |
| `ast.Name` | Variable reference | `x` |
| `ast.Constant` | Literal value | `42`, `"hello"` |
| `ast.Import` | Import statement | `import os` |
| `ast.ImportFrom` | From-import | `from os import path` |
| `ast.BinOp` | Binary operation | `a + b` |
| `ast.Return` | Return statement | `return x` |

While `ast.walk()` is fine for simple searches, the `ast.NodeVisitor` class provides a much more structured approach. You subclass it and define `visit_<NodeType>` methods for each node type you want to handle. The following example finds all function definitions, including async ones.

```python
import ast

class FunctionFinder(ast.NodeVisitor):
    def __init__(self):
        self.functions = []

    def visit_FunctionDef(self, node):
        self.functions.append(node.name)
        self.generic_visit(node)  # Continue visiting children

    def visit_AsyncFunctionDef(self, node):
        self.functions.append(f"async {node.name}")
        self.generic_visit(node)

code = """
def greet(name):
    return f"Hello, {name}"

async def fetch_data(url):
    pass

def main():
    greet("Alice")
"""

finder = FunctionFinder()
finder.visit(ast.parse(code))
print(finder.functions)  # ['greet', 'async fetch_data', 'main']
```

The visitor dispatches each node to the matching `visit_*` method, and `generic_visit()` ensures that child nodes (like nested function definitions) are also traversed.

You can combine multiple visitor methods into a single class to build powerful analysis tools. The following example extracts all imports from a module, handling both `import` and `from ... import` statements.

```python
class ImportAnalyzer(ast.NodeVisitor):
    def __init__(self):
        self.imports = []

    def visit_Import(self, node):
        for alias in node.names:
            self.imports.append(alias.name)
        self.generic_visit(node)

    def visit_ImportFrom(self, node):
        for alias in node.names:
            self.imports.append(f"{node.module}.{alias.name}")
        self.generic_visit(node)

code = """
import os
import sys
from pathlib import Path
from collections import defaultdict, Counter
"""

analyzer = ImportAnalyzer()
analyzer.visit(ast.parse(code))
print(analyzer.imports)
# ['os', 'sys', 'pathlib.Path', 'collections.defaultdict', 'collections.Counter']
```

This pattern scales naturally -- add more `visit_*` methods to handle additional node types without changing the traversal logic.

One critical detail deserves special attention: the `generic_visit()` method. If you define a `visit_*` method but do not call `self.generic_visit(node)`, the visitor will not descend into that node's children. This matters whenever nodes can be nested.

```python
class CallCounter(ast.NodeVisitor):
    def __init__(self):
        self.count = 0

    def visit_Call(self, node):
        self.count += 1
        self.generic_visit(node)  # Don't forget! Nested calls exist.

# f(g(x)) has 2 Call nodes — generic_visit ensures both are counted.
```

Without the `generic_visit()` call, the inner `g(x)` call would never be visited, and the count would be wrong.

## Common Pitfalls

- **Forgetting `generic_visit()`**: This is the most common mistake. If your `visit_*` method does not call `self.generic_visit(node)`, child nodes are silently skipped. Nested function definitions, nested calls, and nested classes will all be missed.
- **Confusing `Name` contexts**: A `Name` node has a `ctx` attribute that is either `Load()`, `Store()`, or `Del()`. The same variable name `x` appears as `Name(id='x', ctx=Store())` on the left side of an assignment and `Name(id='x', ctx=Load())` on the right side. Ignoring the context can lead to incorrect analysis.
- **Assuming node field names**: Different node types have different field names. For example, `FunctionDef` uses `node.name` but `Assign` uses `node.targets`. Always check `ast.dump()` or the documentation to confirm field names before accessing them.

## Best Practices

- **Start with `ast.dump()` before writing visitors**: Always dump the tree for a representative code sample first. This lets you see the exact node types and field names you need to handle.
- **Keep visitors focused on one concern**: Rather than building one massive visitor that does everything, create small, single-purpose visitors (one for imports, one for function signatures, one for complexity metrics) and compose them.
- **Use `isinstance()` checks for polymorphic nodes**: Some constructs like `ast.Name` appear in many contexts. Use `isinstance(node.ctx, ast.Store)` to disambiguate rather than trying to infer from position in the tree.

## Summary

- `ast.dump()` is your primary debugging tool for understanding tree structure -- use it before writing any visitor code.
- Python defines specific node types for every syntactic construct, from `FunctionDef` and `ClassDef` to `Call`, `Name`, and `Constant`.
- The `NodeVisitor` pattern provides a clean, extensible way to traverse the tree by defining `visit_<NodeType>` methods for each node type you care about.
- Always call `self.generic_visit(node)` in your visitor methods to ensure child nodes are traversed.
- Compose small, focused visitors rather than building monolithic analyzers.

## Code Examples

**Finding all function calls with NodeVisitor**

```python
import ast

class CallFinder(ast.NodeVisitor):
    """Find all function/method calls in source code."""
    def __init__(self):
        self.calls = []

    def visit_Call(self, node):
        # Extract the function name being called
        if isinstance(node.func, ast.Name):
            self.calls.append(node.func.id)
        elif isinstance(node.func, ast.Attribute):
            self.calls.append(node.func.attr)
        self.generic_visit(node)

code = """
results = []
for item in get_items():
    results.append(item.strip())
print(len(results))
"""

finder = CallFinder()
finder.visit(ast.parse(code))
print(finder.calls)  # ['get_items', 'append', 'strip', 'print', 'len']
```


## Resources

- [ast — Abstract Syntax Trees (Python docs)](https://docs.python.org/3.14/library/ast.html) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*