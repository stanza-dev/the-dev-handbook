---
source_course: "python"
source_lesson: "python-itertools-intro"
---

# Itertools for Advanced Iteration

## Introduction
The `itertools` module is Python's Swiss Army knife for iteration. It provides memory-efficient tools for infinite sequences, combinatorics, grouping, and slicing -- all without materializing full lists in memory.

## Key Concepts
- **Infinite iterators**: `count`, `cycle`, `repeat` -- produce values endlessly (always pair with a stopping condition).
- **Finite iterators**: `chain`, `islice`, `takewhile`, `dropwhile` -- compose and slice existing iterables.
- **Combinatoric iterators**: `permutations`, `combinations`, `product` -- generate ordered/unordered selections.
- **Grouping**: `groupby` -- groups consecutive elements by a key function.

## Real World Context
Data pipelines and ETL scripts use `chain` to concatenate file streams, `islice` to paginate results, and `groupby` to aggregate records. Combinatoric functions power test-case generators and scheduling algorithms. Because itertools operates lazily, it handles datasets that would not fit in memory as a list.

## Deep Dive

### Infinite Iterators

```python
from itertools import count, cycle, repeat

# count(start, step) - infinite counter
for i in count(10, 2):  # 10, 12, 14, ...
    if i > 20: break

# cycle(iterable) - repeat infinitely
colors = cycle(["red", "green", "blue"])

# repeat(elem, n) - repeat n times
list(repeat("x", 3))  # ['x', 'x', 'x']
```

### Finite Iterators

```python
from itertools import chain, islice, takewhile, dropwhile

# chain - combine iterables
list(chain([1, 2], [3, 4]))  # [1, 2, 3, 4]

# islice - slice an iterator
list(islice(range(100), 5, 10))  # [5, 6, 7, 8, 9]

# takewhile - take while condition is true
list(takewhile(lambda x: x < 5, [1, 3, 5, 7, 2]))
# [1, 3]

# dropwhile - skip while condition is true
list(dropwhile(lambda x: x < 5, [1, 3, 5, 7, 2]))
# [5, 7, 2]
```

### Combinatorics

```python
from itertools import permutations, combinations, product

# permutations - all orderings
list(permutations([1, 2, 3], 2))
# [(1,2), (1,3), (2,1), (2,3), (3,1), (3,2)]

# combinations - unordered selections
list(combinations([1, 2, 3], 2))
# [(1,2), (1,3), (2,3)]

# product - cartesian product
list(product([1, 2], ['a', 'b']))
# [(1,'a'), (1,'b'), (2,'a'), (2,'b')]
```

### Grouping

```python
from itertools import groupby

data = [("a", 1), ("a", 2), ("b", 3)]
for key, group in groupby(data, key=lambda x: x[0]):
    print(key, list(group))
# a [('a', 1), ('a', 2)]
# b [('b', 3)]
```

## Common Pitfalls
1. **Forgetting that `groupby` requires sorted input** -- `groupby` groups consecutive elements, not all elements with the same key. Sort the data by the key first: `groupby(sorted(data, key=func), key=func)`.
2. **Exhausting an infinite iterator** -- Calling `list(count())` will run until you run out of memory. Always combine infinite iterators with `islice`, `takewhile`, or a `break` condition.
3. **Re-iterating over a consumed iterator** -- Itertools objects are single-pass. Once exhausted, they produce no more values. Call the function again or use `itertools.tee` if you need multiple passes.

## Best Practices
1. **Use `chain.from_iterable` when you have a list of lists** -- `chain.from_iterable(list_of_lists)` is more efficient than unpacking with `chain(*list_of_lists)`.
2. **Prefer `islice` over list slicing for large iterators** -- It avoids materializing the entire sequence in memory.

## Summary
- `itertools` provides lazy, memory-efficient tools for infinite sequences, slicing, combinatorics, and grouping.
- Infinite iterators (`count`, `cycle`, `repeat`) must always be bounded by a stopping condition.
- `groupby` requires data to be sorted by the grouping key.
- All itertools objects are single-pass; re-create them if you need to iterate again.
- Use `chain.from_iterable` and `islice` for efficient processing of large or nested datasets.

## Code Examples

**Modern itertools additions**

```python
from itertools import accumulate, pairwise

# accumulate - running totals
list(accumulate([1, 2, 3, 4]))
# [1, 3, 6, 10]

# pairwise (3.10+) - consecutive pairs
list(pairwise([1, 2, 3, 4]))
# [(1, 2), (2, 3), (3, 4)]

# batched (3.12+) - group into chunks
from itertools import batched
list(batched(range(10), 3))
# [(0,1,2), (3,4,5), (6,7,8), (9,)]
```


## Resources

- [itertools Module](https://docs.python.org/3.14/library/itertools.html) — Official Python 3.14 reference for the itertools module with recipes and examples

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*