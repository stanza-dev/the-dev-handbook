---
source_course: "python"
source_lesson: "python-structural-pattern-matching"
---

# Match / Case (Python 3.10+)

Structural Pattern Matching is Python's most powerful control flow feature. It's not just a switch statement—it matches and destructures data based on its shape.

## Basic Syntax

```python
match value:
    case pattern1:
        # action for pattern1
    case pattern2:
        # action for pattern2
    case _:
        # default case (wildcard)
```

## Literal Patterns

```python
match status_code:
    case 200:
        return "OK"
    case 404:
        return "Not Found"
    case 500 | 502 | 503:  # OR patterns
        return "Server Error"
    case _:
        return "Unknown"
```

## Sequence Patterns

```python
match command:
    case ["quit"]:
        quit()
    case ["load", filename]:
        load(filename)
    case ["save", filename, *options]:
        save(filename, options)
    case [first, *middle, last]:
        print(f"First: {first}, Last: {last}")
```

## Mapping Patterns

```python
match response:
    case {"status": 200, "data": data}:
        process(data)
    case {"status": 404}:
        print("Not found")
    case {"error": message}:
        print(f"Error: {message}")
```

## Class Patterns

```python
match point:
    case Point(x=0, y=0):
        print("Origin")
    case Point(x=0, y=y):
        print(f"On Y-axis at {y}")
    case Point(x=x, y=0):
        print(f"On X-axis at {x}")
    case Point(x=x, y=y):
        print(f"Point at ({x}, {y})")
```

## Guards

Add conditions with `if`:

```python
match point:
    case Point(x=x, y=y) if x == y:
        print("On diagonal")
    case Point(x=x, y=y):
        print(f"Point at ({x}, {y})")
```

## Code Examples

**Pattern matching for API responses**

```python
# Real-world example: API response handling
def handle_response(response):
    match response:
        case {"status": "success", "data": {"users": users}}:
            return [User(**u) for u in users]
        case {"status": "error", "code": code, "message": msg}:
            raise APIError(code, msg)
        case {"status": "pending", "retry_after": seconds}:
            time.sleep(seconds)
            return retry()
        case _:
            raise ValueError("Unexpected response format")
```


## Resources

- [PEP 634: Structural Pattern Matching](https://peps.python.org/pep-0634/) — Specification for the match statement

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*