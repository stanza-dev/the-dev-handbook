---
source_course: "python"
source_lesson: "python-variables-walrus-fstrings"
---

# Variables, Scopes & Assignment Expressions

## Introduction
Variables in Python are not boxes that hold values -- they are name tags that point to objects in memory. This lesson explains how assignment works, how Python resolves names through the LEGB rule, and how the walrus operator lets you assign and test in a single expression.

## Key Concepts
- **Variable assignment**: Binding a name to an object reference, not copying a value.
- **LEGB rule**: The lookup order Python uses to resolve names -- Local, Enclosing, Global, Built-in.
- **`global` / `nonlocal`**: Keywords that let you write to variables in outer scopes.
- **Walrus operator (`:=`)**: An assignment expression (Python 3.8+) that assigns and returns a value in one step.

## Real World Context
Scope bugs are among the hardest to diagnose. A function that accidentally shadows a global variable or mutates a closure variable without `nonlocal` can behave correctly in unit tests yet fail in production. Understanding LEGB and the walrus operator helps you write concise, bug-free conditionals and comprehensions.

## Deep Dive

### Variable Assignment

Python variables are references to objects. Assignment binds a name to an object.

```python
x = 42        # x refers to an int object
y = x         # y refers to the same object
x = "hello"   # x now refers to a string, y still refers to 42
```

### Multiple Assignment

```python
a, b, c = 1, 2, 3
x = y = z = 0  # All refer to the same object
a, b = b, a    # Swap values
```

### The LEGB Rule

Python resolves variable names in this order:

1. **L**ocal -- Inside the current function
2. **E**nclosing -- In outer functions (for nested functions)
3. **G**lobal -- At the module level
4. **B**uilt-in -- Python's built-in names

```python
x = "global"

def outer():
    x = "enclosing"
    
    def inner():
        x = "local"
        print(x)  # "local"
    
    inner()
    print(x)  # "enclosing"

outer()
print(x)  # "global"
```

### The `global` and `nonlocal` Keywords

```python
count = 0

def increment():
    global count
    count += 1

def outer():
    x = 10
    def inner():
        nonlocal x
        x += 1
    inner()
    print(x)  # 11
```

### The Walrus Operator `:=` (Assignment Expression)

Introduced in Python 3.8, it assigns and returns a value in one expression.

```python
# Without walrus
data = get_data()
if data:
    process(data)

# With walrus
if (data := get_data()):
    process(data)

# In list comprehensions
results = [y for x in items if (y := expensive(x)) > 0]
```

## Common Pitfalls
1. **Forgetting `global` or `nonlocal` when modifying outer variables** -- Without these keywords, assigning to a variable inside a function creates a new local variable instead of modifying the outer one, leading to `UnboundLocalError`.
2. **Overusing the walrus operator** -- `:=` is powerful for `while` loops and comprehension filters, but nesting multiple walrus assignments in one expression hurts readability.
3. **Shadowing built-in names** -- Naming a variable `list`, `dict`, or `id` hides the built-in and causes hard-to-find bugs later.

## Best Practices
1. **Minimize use of `global`** -- Global mutable state makes code hard to test and reason about. Pass values as function arguments instead.
2. **Use the walrus operator for the "compute-then-check" pattern** -- It shines in `while (line := f.readline()):` and `[y for x in data if (y := transform(x)) is not None]`.

## Summary
- Python variables are name-to-object bindings, not value containers.
- The LEGB rule determines name resolution order: Local, Enclosing, Global, Built-in.
- Use `global` and `nonlocal` sparingly to modify variables in outer scopes.
- The walrus operator `:=` assigns and returns a value in one expression, ideal for while loops and comprehension filters.
- Avoid shadowing built-in names like `list`, `dict`, and `id`.

## Code Examples

**Practical walrus operator usage**

```python
# Walrus operator in a while loop
while (line := input("Enter text: ")) != "quit":
    print(f"You entered: {line}")

# Walrus in list comprehension filtering
data = [1, 2, 3, 4, 5]
results = [squared for x in data if (squared := x**2) > 10]
print(results)  # [16, 25]
```


## Resources

- [Assignment Statements](https://docs.python.org/3.14/reference/simple_stmts.html#assignment-statements) — Official Python 3.14 reference for assignment statements, including augmented and walrus operator

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*