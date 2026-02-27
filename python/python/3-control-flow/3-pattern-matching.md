---
source_course: "python"
source_lesson: "python-structural-pattern-matching"
---

# Structural Pattern Matching

## Introduction
Structural Pattern Matching, introduced in Python 3.10, goes far beyond a simple switch statement. It lets you match values, destructure sequences and mappings, and inspect object shapes -- all in a clean, declarative syntax.

## Key Concepts
- **`match` / `case`**: The statement that dispatches on the shape and value of data.
- **Literal pattern**: Matches exact values like `200`, `"error"`, or combined with `|` for OR.
- **Sequence pattern**: Destructures lists and tuples, e.g., `[first, *rest]`.
- **Mapping pattern**: Matches dict-like structures by key.
- **Class pattern**: Matches objects by type and attribute values.
- **Guard**: An `if` clause that adds an extra condition to a `case`.

## Real World Context
Pattern matching simplifies code that dispatches on the shape of data -- parsing CLI commands, handling API responses with varying structures, or processing AST nodes. Without it, you would write chains of `isinstance` checks and dictionary key lookups. With it, each case reads like a specification of the data it handles.

## Deep Dive

### Basic Syntax

```python
match value:
    case pattern1:
        # action for pattern1
    case pattern2:
        # action for pattern2
    case _:
        # default case (wildcard)
```

### Literal Patterns

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

### Sequence Patterns

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

### Mapping Patterns

```python
match response:
    case {"status": 200, "data": data}:
        process(data)
    case {"status": 404}:
        print("Not found")
    case {"error": message}:
        print(f"Error: {message}")
```

### Class Patterns

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

### Guards

Add conditions with `if`:

```python
match point:
    case Point(x=x, y=y) if x == y:
        print("On diagonal")
    case Point(x=x, y=y):
        print(f"Point at ({x}, {y})")
```

## Common Pitfalls
1. **Using a bare name as a literal pattern** -- `case status:` captures the value into a variable named `status` rather than matching a constant. Use dotted names (`case HTTPStatus.OK:`) or literal values (`case 200:`) for matching.
2. **Forgetting the wildcard `_` case** -- Without a default case, unmatched values silently fall through with no action, which may hide bugs.
3. **Putting a more general pattern before a specific one** -- Cases are checked top to bottom. A broad pattern like `case [*items]:` before `case ["quit"]:` will match first and prevent the specific case from ever running.

## Best Practices
1. **Order cases from most specific to least specific** -- This ensures specialized patterns match before generic catch-alls.
2. **Use guards sparingly** -- If a guard becomes complex, extract it into a helper function for readability.

## Summary
- `match`/`case` matches and destructures data based on its shape, going far beyond a switch statement.
- Literal, sequence, mapping, and class patterns cover nearly every data dispatching need.
- Guards add extra conditions to patterns with an `if` clause.
- Always include a wildcard `_` case to handle unexpected data.
- Order cases from most specific to most general to avoid shadowing.

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

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*