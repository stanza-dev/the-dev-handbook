---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-compile-function"
---

# The compile() Function & Code Objects

## Introduction

Before Python can run any source code, it must first compile that code into a bytecode representation called a **code object**. The `compile()` built-in gives you explicit control over this compilation step, letting you separate parsing from execution for better performance, earlier error detection, and powerful introspection capabilities.

## Key Concepts

- **Code object**: Python's internal representation of compiled bytecode, produced by `compile()` and normally hidden inside functions and modules.
- **`compile()`**: A built-in function that takes a source string (or AST node) and returns a code object that can be passed to `exec()` or `eval()`.
- **Compilation mode**: One of `'exec'` (statements), `'eval'` (single expression), or `'single'` (interactive statement), which tells the compiler what kind of code to expect.
- **`dis` module**: The standard library disassembler that converts code objects into human-readable bytecode instructions.
- **Bytecode**: The low-level instruction set that the CPython virtual machine executes, stored in a code object's `co_code` attribute.

## Real World Context

When you build a system that evaluates user-supplied formulas thousands of times -- think a spreadsheet engine or a rule-based pricing calculator -- recompiling the formula string on every call wastes significant CPU time. By calling `compile()` once and reusing the resulting code object, you get the same result with a fraction of the overhead. This is the exact pattern used internally by Python's `dataclasses`, `namedtuple`, and many template engines.

## Deep Dive

### compile() Basics

The full signature of `compile()` is shown below:

```python
compile(source, filename, mode, flags=0, dont_inherit=False, optimize=-1)
```

The three most important parameters are: **source** (a string of Python code or an AST node), **filename** (used in error messages and tracebacks -- use `"<string>"` for dynamic code), and **mode** (determines what kind of code is expected).

### The Three Modes

#### 'exec' -- Sequence of Statements

Use `'exec'` mode for module-level code that contains definitions, loops, assignments, or any combination of statements. The following example compiles a block of arithmetic and prints the result:

```python
code_obj = compile("""
x = 10
y = 20
result = x + y
print(result)
""", "<string>", "exec")

exec(code_obj)  # 30
```

The compiled code object can be passed to `exec()` just like a raw string, but without the cost of re-parsing.

#### 'eval' -- Single Expression

Use `'eval'` mode when your source is a single expression that produces a value. No statements are allowed. This example compiles a power expression and evaluates it:

```python
code_obj = compile("2 ** 10", "<string>", "eval")
result = eval(code_obj)
print(result)  # 1024
```

Because the code object is returned by `compile()`, you can store it and call `eval()` on it repeatedly with different namespaces.

#### 'single' -- Interactive Statement

The `'single'` mode mimics the interactive Python REPL: it compiles a single statement and auto-prints expression results. This is primarily useful for building custom interactive shells:

```python
code_obj = compile("3 + 4", "<string>", "single")
exec(code_obj)  # Prints: 7 (auto-prints like the interactive shell)
```

Notice that `exec()` is used even with `'single'` mode -- the auto-printing behavior is baked into the compiled code object itself.

### Why Use compile() Instead of Passing Strings?

There are three compelling reasons to pre-compile your dynamic code. First, **performance**: you compile once and execute many times without re-parsing. Second, **early error detection**: syntax errors are caught at compile time rather than at execution time. Third, **inspection**: you can examine the code object before deciding whether to execute it. The following example demonstrates the performance benefit of compile-once-run-many:

```python
# Compile once, run many times
code_obj = compile("x * 2 + 1", "<formula>", "eval")

for x_val in range(5):
    result = eval(code_obj, {"x": x_val})
    print(f"x={x_val} -> {result}")
# x=0 -> 1, x=1 -> 3, x=2 -> 5, x=3 -> 7, x=4 -> 9
```

Each iteration reuses the already-compiled code object, skipping the parsing and compilation overhead entirely.

### Exploring Code Objects

Code objects carry rich metadata about the compiled code. You can inspect their attributes to understand variable usage, constants, and structure. The example below compiles a function definition and then drills into the function's own code object:

```python
source = """
def add(a, b):
    total = a + b
    return total
"""
code_obj = compile(source, "example.py", "exec")

# The module-level code object
print(code_obj.co_consts)    # Constants used in the code
print(code_obj.co_names)     # Names referenced

# Get the function's code object from constants
func_code = code_obj.co_consts[0]
print(func_code.co_varnames) # ('a', 'b', 'total')
print(func_code.co_consts)   # (None,) -- literal constants
```

The nested code object for `add` reveals its local variables (`a`, `b`, `total`) and any literal constants used in the function body. Here are the most commonly inspected code object attributes:

| Attribute | Description |
|-----------|-------------|
| `co_varnames` | Local variable names (including parameters) |
| `co_consts` | Literal constants used in the code |
| `co_names` | Names used (global/attribute lookups) |
| `co_code` | Raw bytecode as bytes |
| `co_filename` | The filename argument from compile() |
| `co_argcount` | Number of positional arguments |

### Bytecode Inspection with dis

The `dis` module from the standard library disassembles code objects into human-readable bytecode instructions. This is invaluable for understanding how Python executes your code under the hood. Here is an example that disassembles a simple multiplication function:

```python
import dis

def multiply(a, b):
    return a * b

dis.dis(multiply)
#   2     LOAD_FAST    0 (a)
#         LOAD_FAST    1 (b)
#         BINARY_OP    5 (*)
#         RETURN_VALUE
```

Each line shows a bytecode instruction: `LOAD_FAST` pushes a local variable onto the stack, `BINARY_OP` performs the multiplication, and `RETURN_VALUE` pops the result and returns it. You can also disassemble code objects created from compiled strings:

```python
code_obj = compile("x + y * 2", "<expr>", "eval")
dis.dis(code_obj)
#   1     LOAD_NAME    0 (x)
#         LOAD_NAME    1 (y)
#         LOAD_CONST   0 (2)
#         BINARY_OP    5 (*)
#         BINARY_OP    0 (+)
#         RETURN_VALUE
```

This output confirms that Python correctly applies operator precedence: `y * 2` is computed before adding `x`. Being able to read bytecode is invaluable for understanding Python internals and optimizing performance-critical code.

## Common Pitfalls

1. **Using the wrong mode.** Passing a multi-line block to `'eval'` mode or a bare expression to `'exec'` mode will raise a `SyntaxError`. Match the mode to your source: `'eval'` for single expressions, `'exec'` for statement blocks.
2. **Ignoring the filename argument.** Passing `"<string>"` everywhere makes tracebacks unhelpful. Use descriptive filenames like `"<generated:ClassName>"` or `f"config://{path}"` so you can trace errors back to their source.
3. **Recompiling on every call.** The whole point of `compile()` is to separate compilation from execution. If you call `compile()` inside a loop that also calls `eval()`, you lose the performance benefit entirely.

## Best Practices

1. **Compile once, execute many.** Store the code object in a variable or cache and reuse it across multiple `eval()` or `exec()` calls to avoid repeated parsing overhead.
2. **Use descriptive filenames.** The `filename` argument appears in tracebacks and error messages. Make it meaningful so that debugging dynamically generated code is straightforward.
3. **Combine with `dis` for debugging.** When dynamically generated code behaves unexpectedly, disassemble the code object with `dis.dis()` to see exactly what bytecode Python produced.

## Summary

- `compile()` converts a source string into a reusable code object, separating parsing from execution.
- The three modes -- `'exec'`, `'eval'`, and `'single'` -- control what kind of Python code the compiler expects.
- Pre-compiling with `compile()` improves performance (compile once, run many), catches syntax errors early, and enables code object inspection.
- Code objects expose attributes like `co_varnames`, `co_consts`, and `co_names` for introspecting compiled code.
- The `dis` module disassembles code objects into readable bytecode, which is essential for understanding Python internals.

## Code Examples

**Compile once, inspect, and execute many times**

```python
import dis

# Compile a formula once
formula = compile("x ** 2 + 2 * x + 1", "<formula>", "eval")

# Inspect the code object
print(f"Constants: {formula.co_consts}")   # (2, 1)
print(f"Names:     {formula.co_names}")    # ('x',)

# Disassemble to see bytecode
print("\nBytecode:")
dis.dis(formula)

# Execute many times efficiently
for val in [0, 1, 2, 3]:
    result = eval(formula, {"x": val})
    print(f"f({val}) = {result}")
# f(0) = 1, f(1) = 4, f(2) = 9, f(3) = 16
```


## Resources

- [undefined](https://docs.python.org/3.14/library/functions.html#compile) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*