---
source_course: "python-architecture"
source_lesson: "python-architecture-property-based-testing"
---

# Property-Based Testing

## Introduction

Traditional example-based tests verify specific input-output pairs: `assert add(2, 2) == 4`. Property-based testing flips this approach — instead of choosing examples, you describe **properties** that must hold for *all* valid inputs, and a framework generates hundreds or thousands of random inputs to verify them.

## Key Concepts

- **Property**: An invariant that should hold true regardless of input (e.g., reversing a list twice yields the original list).
- **Strategy**: A description of how to generate random test data (integers, strings, lists, custom objects).
- **Shrinking**: When a failing case is found, the framework automatically simplifies it to the minimal reproducing example.
- **Stateful testing**: Testing sequences of operations against a model to find state-dependent bugs.

## Real World Context

Property-based testing excels at finding edge cases humans overlook: empty collections, negative numbers, Unicode surrogates, integer overflow boundaries. Companies like Volvo and Ericsson use it extensively for safety-critical systems. The Hypothesis library for Python has found bugs in major projects including NumPy, Pandas, and the CPython standard library itself.

## Deep Dive

### The Hypothesis Library

Hypothesis is the standard property-based testing library for Python. It integrates with pytest and unittest.

```python
from hypothesis import given, settings, assume
from hypothesis import strategies as st

@given(st.lists(st.integers()))
def test_sorted_is_idempotent(xs):
    """Sorting a sorted list should not change it."""
    assert sorted(sorted(xs)) == sorted(xs)

@given(st.integers(), st.integers())
def test_addition_is_commutative(a, b):
    """Addition order should not matter."""
    assert a + b == b + a

@given(st.text())
def test_encode_decode_roundtrip(s):
    """Encoding then decoding UTF-8 should return the original."""
    assert s.encode('utf-8').decode('utf-8') == s
```

### Custom Strategies

```python
from hypothesis import strategies as st
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int

# Build a strategy that generates User objects
users = st.builds(
    User,
    name=st.text(min_size=1, max_size=50),
    age=st.integers(min_value=0, max_value=150)
)

@given(users)
def test_user_display_name(user):
    display = f"{user.name} (age {user.age})"
    assert user.name in display
    assert str(user.age) in display
```

### Shrinking in Action

When Hypothesis finds a failing input like `[1073741824, -3, 0, 99, 42]`, it automatically shrinks it to the simplest failing case, such as `[1073741824]`. This makes debugging far easier than staring at a random 50-element list.

### Settings and Profiles

```python
from hypothesis import settings, Phase

# Run more examples in CI
@settings(max_examples=1000)
@given(st.integers())
def test_thorough(n):
    assert abs(n) >= 0

# Skip shrinking for faster feedback
@settings(phases=[Phase.explicit, Phase.generate])
@given(st.floats(allow_nan=False))
def test_quick(x):
    assert x * 0 == 0
```

## Common Pitfalls

1. **Testing implementation, not properties**: `assert my_sort(xs) == sorted(xs)` just re-tests Python's sort. Instead, test properties like "output is sorted" and "output is a permutation of input".
2. **Overly strict `assume()` calls**: Using `assume()` to filter too many inputs makes tests slow and reduces coverage. Use targeted strategies instead.
3. **Non-deterministic tests**: If your code uses `random` or `datetime.now()`, inject those as dependencies so Hypothesis controls them.
4. **Ignoring the Hypothesis database**: Hypothesis remembers previously failing examples in `.hypothesis/`. Commit this directory or your CI will lose regression coverage.

## Best Practices

- Start by identifying **universal properties**: idempotency, commutativity, round-trip encoding, invariants.
- Use `st.from_type()` to auto-generate data from type hints.
- Combine property-based tests with a few explicit example-based tests for documentation.
- Set `@settings(max_examples=500)` in CI for thorough testing.
- Use `@example(...)` to pin known edge cases alongside generated ones.

## Summary

Property-based testing complements example-based testing by exploring the input space automatically. Define properties, let Hypothesis generate inputs, and benefit from automatic shrinking when failures occur. It is particularly valuable for algorithmic code, serialization, and any function with clear mathematical properties.

## Code Examples

**Round-trip property test for run-length encoding**

```python
from hypothesis import given, example
from hypothesis import strategies as st

def encode(text: str) -> list[int]:
    """Run-length encode a string."""
    if not text:
        return []
    result = []
    count = 1
    for i in range(1, len(text)):
        if text[i] == text[i - 1]:
            count += 1
        else:
            result.extend([ord(text[i - 1]), count])
            count = 1
    result.extend([ord(text[-1]), count])
    return result

def decode(data: list[int]) -> str:
    """Decode run-length encoded data."""
    return ''.join(chr(data[i]) * data[i + 1] for i in range(0, len(data), 2))

# Property: encode then decode is a round-trip
@given(st.text(alphabet=st.characters(whitelist_categories=('L', 'N')), min_size=0, max_size=100))
@example('')
@example('aaa')
def test_roundtrip(text):
    assert decode(encode(text)) == text
```

**Multiple properties tested on a sort function**

```python
from hypothesis import given
from hypothesis import strategies as st

def my_sort(xs: list[int]) -> list[int]:
    return sorted(xs)

@given(st.lists(st.integers()))
def test_sort_preserves_length(xs):
    assert len(my_sort(xs)) == len(xs)

@given(st.lists(st.integers()))
def test_sort_is_ordered(xs):
    result = my_sort(xs)
    for i in range(len(result) - 1):
        assert result[i] <= result[i + 1]

@given(st.lists(st.integers()))
def test_sort_is_idempotent(xs):
    assert my_sort(my_sort(xs)) == my_sort(xs)

@given(st.lists(st.integers()))
def test_sort_preserves_elements(xs):
    from collections import Counter
    assert Counter(my_sort(xs)) == Counter(xs)
```


## Resources

- [Hypothesis Documentation](https://hypothesis.readthedocs.io/en/latest/) — Official Hypothesis library documentation
- [Property-based testing with Hypothesis](https://hypothesis.readthedocs.io/en/latest/quickstart.html) — Quick-start guide for writing property-based tests with Hypothesis

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*