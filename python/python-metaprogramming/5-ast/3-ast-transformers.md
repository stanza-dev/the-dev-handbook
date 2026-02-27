---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-ast-transformers"
---

# AST Transformers

## Introduction

While `NodeVisitor` lets you read and analyze an AST, `NodeTransformer` lets you rewrite it. This is the mechanism behind tools that programmatically modify Python code -- from replacing deprecated API calls across an entire codebase to injecting instrumentation into every function. If visitors are read-only lenses, transformers are the editing tools that make AST manipulation truly powerful.

## Key Concepts

- **`ast.NodeTransformer`**: A subclass of `NodeVisitor` where each `visit_*` method returns a node (to keep or replace it) or `None` (to delete it).
- **`ast.fix_missing_locations()`**: Copies line number and column offset information from parent nodes to newly created nodes that lack this metadata.
- **Node replacement**: Returning a different node from a `visit_*` method substitutes it in the tree.
- **Node deletion**: Returning `None` from a `visit_*` method removes that node from its parent's children.
- **Node insertion**: Returning a list of nodes from a `visit_*` method replaces the single node with multiple nodes.

## Real World Context

Consider a large codebase that needs to migrate from `print()` statements to a proper logging framework, or a security tool that strips all `assert` statements before deploying to production. Doing this with find-and-replace is fragile and error-prone -- it cannot distinguish `print` the function call from `print` as a variable name or string literal. AST transformers operate on the parsed structure of the code, so they only modify the exact constructs you target, making automated code rewrites safe and precise.

## Deep Dive

`ast.NodeTransformer` is a subclass of `NodeVisitor` with one key difference: each `visit_*` method is expected to return a value. You can return the original node to keep it unchanged, return a new node to replace it, or return `None` to delete the node from the tree.

The following example doubles every numeric constant in the code. It creates a new `Constant` node with the doubled value for numbers, and returns the original node unchanged for strings and other non-numeric constants.

```python
import ast

class DoubleConstants(ast.NodeTransformer):
    """Double every numeric constant in the code."""
    def visit_Constant(self, node):
        if isinstance(node.value, (int, float)):
            return ast.Constant(value=node.value * 2)
        return node  # Leave non-numeric constants unchanged

code = "x = 5 + 10"
tree = ast.parse(code)
new_tree = DoubleConstants().visit(tree)

print(ast.dump(new_tree, indent=2))
# x = 10 + 20 (constants doubled)
```

The transformer visits every `Constant` node and decides whether to replace it. Notice that non-numeric constants (like strings) pass through untouched because we return the original `node`.

When you create new AST nodes, they do not have line number or column offset metadata. Python's `compile()` function requires this information, so you need to call `ast.fix_missing_locations()` to propagate location data from parent nodes to any newly created children.

```python
tree = ast.parse("x = 5 + 10")
new_tree = DoubleConstants().visit(tree)

# Without this, compile() raises: TypeError: required field 'lineno' missing
ast.fix_missing_locations(new_tree)

code_obj = compile(new_tree, filename="<ast>", mode="exec")
exec(code_obj)  # Now works!
```

Forgetting this step is one of the most common sources of confusing errors when working with AST transformers.

The real power of transformers comes from the complete pipeline: parse source code, transform the tree, fix locations, compile, and execute. The following example replaces all `print()` calls with calls to a custom `log()` function.

```python
import ast

class PrintToLog(ast.NodeTransformer):
    """Replace all print() calls with log() calls."""
    def visit_Call(self, node):
        self.generic_visit(node)  # Transform children first
        if isinstance(node.func, ast.Name) and node.func.id == 'print':
            node.func.id = 'log'
        return node

def log(*args):
    import sys
    sys.stderr.write("LOG: " + " ".join(str(a) for a in args) + "\n")

source = """
print("Starting")
result = 42
print("Result:", result)
"""

tree = ast.parse(source)
new_tree = PrintToLog().visit(tree)
ast.fix_missing_locations(new_tree)

code_obj = compile(new_tree, filename="<transformed>", mode="exec")
exec(code_obj, {"log": log})  # Inject our log function
# LOG: Starting
# LOG: Result: 42
```

This demonstrates the full round trip: the source code is parsed, transformed, recompiled, and executed with a custom namespace that provides the `log` function.

Sometimes you want to remove nodes entirely rather than replace them. Returning `None` from a `visit_*` method deletes that node from the tree. This is useful for stripping debug code, assert statements, or other constructs before deployment.

```python
class RemoveAsserts(ast.NodeTransformer):
    """Strip all assert statements from code."""
    def visit_Assert(self, node):
        return None  # Delete the node

code = """
x = 10
assert x > 0, "x must be positive"
print(x)
assert isinstance(x, int)
"""

tree = ast.parse(code)
cleaned = RemoveAsserts().visit(tree)
ast.fix_missing_locations(cleaned)
# Only 'x = 10' and 'print(x)' remain
```

After the transformation, both `assert` statements are gone and only the assignment and print call remain in the tree.

You can also insert new nodes by returning a list from a `visit_*` method. When you return a list of nodes, they replace the single original node in the parent's body. This is how you inject instrumentation or logging into existing code.

```python
class AddDebugPrint(ast.NodeTransformer):
    """Add a debug print before every assignment."""
    def visit_Assign(self, node):
        target_name = ast.dump(node.targets[0])
        debug_print = ast.parse(f'print("Assigning to: {target_name}")').body[0]
        return [debug_print, node]  # Insert print before assignment
```

The returned list `[debug_print, node]` tells the transformer to place the debug print statement immediately before the original assignment in the parent block.

Finally, just like with `NodeVisitor`, always call `self.generic_visit(node)` when your transformer handles nodes that can have children. This ensures child nodes are also transformed before you process the parent.

```python
def visit_Call(self, node):
    self.generic_visit(node)  # Transform children first
    # Then transform this node
    return node
```

Calling `generic_visit` before your own logic is the typical pattern for transformers, since you usually want child transformations to complete before deciding how to handle the parent.

## Common Pitfalls

- **Forgetting `ast.fix_missing_locations()`**: Newly created nodes lack `lineno` and `col_offset` attributes. Without calling `ast.fix_missing_locations()` on the transformed tree, `compile()` will raise a `TypeError` about missing required fields.
- **Mutating nodes in place without returning them**: Unlike `NodeVisitor`, `NodeTransformer` relies on the return value of each `visit_*` method. If you modify a node but forget to `return node`, the node is effectively deleted (treated as `return None`).
- **Skipping `generic_visit()` on compound nodes**: If your transformer handles `Call` or `If` nodes but does not call `self.generic_visit(node)`, nested nodes inside those constructs will not be transformed, leading to partial or inconsistent rewrites.

## Best Practices

- **Call `generic_visit()` before your own logic**: Transform children first, then process the parent. This bottom-up approach ensures that the subtree is already in its final form when you make decisions about the parent node.
- **Use `ast.unparse()` to verify transformations**: After transforming a tree, `ast.unparse(new_tree)` converts it back to source code so you can visually confirm the rewrite is correct before compiling and executing.
- **Keep transformers small and composable**: Rather than building one transformer that handles many node types, create focused transformers for each transformation and chain them together. This makes each one easier to test and debug.

## Summary

- `ast.NodeTransformer` extends `NodeVisitor` to enable tree modification: return a new node to replace, `None` to delete, or a list to insert multiple nodes.
- Always call `ast.fix_missing_locations()` on the transformed tree before compiling, since newly created nodes lack required location metadata.
- The full transformation pipeline is: parse, transform, fix locations, compile, execute.
- Returning `None` removes a node; returning a list of nodes replaces one node with many, enabling code injection.
- Call `self.generic_visit(node)` before your own transformation logic to ensure child nodes are processed first.

## Code Examples

**Renaming variables with NodeTransformer**

```python
import ast

class NameReplacer(ast.NodeTransformer):
    """Replace variable references in code."""
    def __init__(self, old_name, new_name):
        self.old_name = old_name
        self.new_name = new_name

    def visit_Name(self, node):
        if node.id == self.old_name:
            return ast.Name(id=self.new_name, ctx=node.ctx)
        return node

# Rename 'tmp' to 'result' everywhere in the code
source = """
tmp = compute()
print(tmp)
return tmp
"""

tree = ast.parse(source)
new_tree = NameReplacer('tmp', 'result').visit(tree)
ast.fix_missing_locations(new_tree)

# Verify the transformation
print(ast.unparse(new_tree))
# result = compute()
# print(result)
# return result
```


## Resources

- [ast — Abstract Syntax Trees (Python docs)](https://docs.python.org/3.14/library/ast.html) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*