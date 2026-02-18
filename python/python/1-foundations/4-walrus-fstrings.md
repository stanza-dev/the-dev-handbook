---
source_course: "python"
source_lesson: "python-variables-walrus-fstrings"
---

# Variables and Scope

## Variable Assignment

Python variables are references to objects. Assignment binds a name to an object.

```python
x = 42        # x refers to an int object
y = x         # y refers to the same object
x = "hello"   # x now refers to a string, y still refers to 42
```

## Multiple Assignment

```python
a, b, c = 1, 2, 3
x = y = z = 0  # All refer to the same object
a, b = b, a    # Swap values
```

## The LEGB Rule

Python resolves variable names in this order:

1. **L**ocal - Inside the current function
2. **E**nclosing - In outer functions (for nested functions)
3. **G**lobal - At the module level
4. **B**uilt-in - Python's built-in names

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

## The `global` and `nonlocal` Keywords

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

## The Walrus Operator `:=` (Assignment Expression)

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*