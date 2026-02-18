---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-ast-parsing"
---

# Abstract Syntax Trees

Python compiles source code into an AST before bytecodes. You can use the `ast` module to parse Python code, analyze it, or even transform it.

```python
import ast

code = "x = 1 + 2"
tree = ast.parse(code)

for node in ast.walk(tree):
    if isinstance(node, ast.Assign):
        print("Found an assignment")
```

## Use Cases
*   Static analysis tools (linters like Ruff/Pylint).
*   Code formatters (Black).
*   Transpilers.

## Code Examples

**AST to execution**

```python
import ast

expr = "1 + 1"
# Parse expression
tree = ast.parse(expr, mode='eval')

# Compile and evaluate
code_obj = compile(tree, filename="<string>", mode="eval")
print(eval(code_obj)) # 2
```


---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*