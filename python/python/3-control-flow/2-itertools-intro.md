---
source_course: "python"
source_lesson: "python-itertools-intro"
---

# The itertools Module

Python's `itertools` module provides efficient iterators for common patterns.

## Infinite Iterators

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

## Finite Iterators

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

## Combinatorics

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

## Grouping

```python
from itertools import groupby

data = [("a", 1), ("a", 2), ("b", 3)]
for key, group in groupby(data, key=lambda x: x[0]):
    print(key, list(group))
# a [('a', 1), ('a', 2)]
# b [('b', 3)]
```

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*