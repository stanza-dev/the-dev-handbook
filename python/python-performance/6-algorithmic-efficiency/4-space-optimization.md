---
source_course: "python-performance"
source_lesson: "python-performance-space-optimization"
---

# Space vs Time Tradeoffs

## Trading Space for Time

```python
# Time: O(n), Space: O(1)
def find_duplicate_slow(nums):
    for i in range(len(nums)):
        for j in range(i+1, len(nums)):
            if nums[i] == nums[j]:
                return nums[i]
    return None

# Time: O(n), Space: O(n)
def find_duplicate_fast(nums):
    seen = set()
    for num in nums:
        if num in seen:
            return num
        seen.add(num)
    return None
```

## Generators for Memory Efficiency

```python
# High memory: Creates entire list
result = [process(x) for x in huge_data]  # All in RAM
for item in result:
    use(item)

# Low memory: Generator
result = (process(x) for x in huge_data)  # Lazy
for item in result:
    use(item)  # One at a time
```

## Streaming vs Batch

```python
# Batch: Load all into memory
with open('huge_file.txt') as f:
    lines = f.readlines()  # All in RAM!
    for line in lines:
        process(line)

# Streaming: One at a time
with open('huge_file.txt') as f:
    for line in f:  # Iterates lazily
        process(line)
```

## Choose Wisely

| Situation | Prefer |
|-----------|--------|
| Small data | Simpler code |
| Large data | Memory efficiency |
| Repeated access | Precompute (memoize) |
| Single pass | Generators |

## Code Examples

**Memory comparison**

```python
import sys

# Compare memory usage
nums = range(1_000_000)

# List: All in memory
lst = [x * 2 for x in nums]
print(f"List: {sys.getsizeof(lst) / 1e6:.1f} MB")

# Generator: Minimal memory
gen = (x * 2 for x in nums)
print(f"Generator: {sys.getsizeof(gen)} bytes")

# Both produce same results!
sum([x * 2 for x in range(1000)]) == sum(x * 2 for x in range(1000))
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*