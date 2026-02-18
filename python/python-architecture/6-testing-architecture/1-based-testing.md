---
source_course: "python-architecture"
source_lesson: "python-architecture-property-based-testing"
---

# Property-Based Testing

Instead of writing example-based tests (`assert add(2, 2) == 4`), you write properties that should always hold true (e.g., `add(a, b) == add(b, a)`). Libraries like `hypothesis` generate thousands of random inputs to find edge cases.

## Mocking
Use `unittest.mock` to replace external systems (APIs, DBs) with controlled objects during tests.

## Code Examples

**Hypothesis concept**

```python
# Example using Hypothesis (pseudo-code)
# @given(st.integers(), st.integers())
# def test_addition_commutative(a, b):
#     assert a + b == b + a
```


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*