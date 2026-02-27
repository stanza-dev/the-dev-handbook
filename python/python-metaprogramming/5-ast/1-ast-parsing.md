---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-ast-parsing"
---

# Parsing Code with AST

## Introduction

Every time Python runs your code, it first converts the source text into an Abstract Syntax Tree -- a structured representation of your program that the interpreter can reason about. The `ast` module gives you direct access to this internal representation, letting you parse code into a tree, walk through its nodes, and even compile it back into executable bytecode. Understanding AST parsing is the foundation for building linters, code formatters, and any tool that needs to understand Python code programmatically.

## Key Concepts

- **Abstract Syntax Tree (AST)**: A tree data structure where each node represents a syntactic construct in your source code, such as assignments, function calls, or operators.
- **`ast.parse()`**: The entry point that converts a string of Python source code into an AST `Module` node.
- **Parsing mode**: Controls what kind of code `ast.parse()` expects -- a full module (`exec`), a single expression (`eval`), or an interactive statement (`single`).
- **`ast.walk()`**: A generator that yields every node in the tree, useful for flat iteration without worrying about tree depth.
- **`compile()`**: Converts an AST back into a code object that Python can execute via `exec()` or `eval()`.

## Real World Context

If you have ever used a linter like Ruff or Flake8, a formatter like Black, or a security scanner that flags dangerous calls to `eval()`, you have benefited from AST parsing. These tools do not work with raw text and regular expressions -- they parse source code into an AST so they can reason about its structure reliably. Understanding how to parse code into an AST is the first step toward building your own analysis and transformation tools.

## Deep Dive

Python compiles source code into an AST before generating bytecode. The `ast` module lets you parse Python source into a tree, inspect it, modify it, and compile it back to executable code.

The simplest way to see this in action is to parse a small snippet and inspect what comes back. The following example parses a simple assignment and shows the types of the resulting tree and its first statement node.

```python
import ast

code = "x = 1 + 2"
tree = ast.parse(code)

# tree is a Module node containing a list of statements
print(type(tree))         # <class 'ast.Module'>
print(type(tree.body[0])) # <class 'ast.Assign'>
```

The `ast.parse()` function returns a `Module` node whose `body` attribute is a list of statement nodes. In this case, the single assignment `x = 1 + 2` becomes an `ast.Assign` node.

`ast.parse()` supports three different modes that control what kind of input it expects. The default `exec` mode parses a full module, `eval` mode parses a single expression, and `single` mode parses one interactive statement.

```python
# mode='exec' (default) — module with statements
tree = ast.parse("x = 1; print(x)")

# mode='eval' — single expression
tree = ast.parse("1 + 2", mode='eval')

# mode='single' — single interactive statement
tree = ast.parse("x = 1", mode='single')
```

Choosing the right mode matters because the resulting tree structure differs: `exec` wraps everything in a `Module`, `eval` wraps the expression in an `Expression` node, and `single` wraps it in an `Interactive` node.

Once you have a tree, you often want to look at every node in it. The `ast.walk()` function yields every node in the tree in no guaranteed order, making it ideal for quick searches.

```python
import ast

code = "x = 1 + 2"
tree = ast.parse(code)

for node in ast.walk(tree):
    if isinstance(node, ast.Assign):
        print("Found an assignment")
    elif isinstance(node, ast.BinOp):
        print("Found a binary operation")
```

This flat iteration style is convenient when you just need to find or count specific node types, without caring about the tree hierarchy.

The real power of AST parsing emerges when you close the loop: parse source into a tree, optionally modify it, then compile and execute it. The following example demonstrates this full round trip.

```python
tree = ast.parse("result = 2 ** 10")
code_obj = compile(tree, filename="<ast>", mode="exec")
namespace = {}
exec(code_obj, namespace)
print(namespace['result'])  # 1024
```

The `compile()` function takes the AST, a filename (used in tracebacks), and a mode that must match the parse mode. The resulting code object can be executed with `exec()` or evaluated with `eval()`, just like any other compiled Python code.

## Common Pitfalls

- **Forgetting the mode mismatch**: If you parse with `mode='eval'` but try to compile with `mode='exec'`, you will get an error. The compile mode must match the parse mode.
- **Assuming `ast.walk()` order**: `ast.walk()` yields nodes in breadth-first order, but the exact traversal order is an implementation detail. Do not rely on nodes appearing in source order -- use `ast.NodeVisitor` if traversal order matters.
- **Parsing invalid syntax**: `ast.parse()` raises `SyntaxError` for invalid code. Always handle this exception when parsing user-provided or dynamically generated source strings.

## Best Practices

- **Use `ast.dump()` for debugging**: When you are unsure what the tree looks like, `ast.dump(tree, indent=2)` prints a readable representation of every node and its fields.
- **Prefer `ast.walk()` for simple searches**: If you just need to find all nodes of a certain type, `ast.walk()` with `isinstance()` checks is simpler and more readable than a full `NodeVisitor` subclass.
- **Always pass a meaningful `filename` to `compile()`**: This string appears in tracebacks and error messages, so using a descriptive name like `"<user-input>"` or the actual file path makes debugging much easier.

## Summary

- Python's `ast` module parses source code into a tree of nodes, each representing a syntactic construct like assignments, expressions, or function calls.
- `ast.parse()` supports three modes: `exec` for modules, `eval` for single expressions, and `single` for interactive statements.
- `ast.walk()` provides flat iteration over all nodes in the tree, ideal for searching and counting.
- The full pipeline -- parse, transform, compile, execute -- lets you programmatically manipulate Python code.
- Real-world tools like linters, formatters, and security scanners all rely on AST parsing as their foundation.

## Code Examples

**AST parsing, compiling, and walking**

```python
import ast

# Parse an expression and evaluate it
expr = "1 + 1"
tree = ast.parse(expr, mode='eval')
code_obj = compile(tree, filename="<string>", mode="eval")
print(eval(code_obj))  # 2

# Parse a module and count assignments
module_code = """
x = 10
y = 20
z = x + y
"""
tree = ast.parse(module_code)
assign_count = sum(1 for node in ast.walk(tree)
                   if isinstance(node, ast.Assign))
print(f"Found {assign_count} assignments")  # Found 3 assignments
```


## Resources

- [ast — Abstract Syntax Trees (Python docs)](https://docs.python.org/3.14/library/ast.html) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*