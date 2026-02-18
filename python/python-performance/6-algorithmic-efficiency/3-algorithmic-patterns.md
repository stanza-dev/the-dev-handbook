---
source_course: "python-performance"
source_lesson: "python-performance-algorithmic-patterns"
---

# Optimization Patterns

## Convert O(n²) to O(n)

```python
# Bad: O(n²) - nested loops
def find_pairs_slow(nums, target):
    pairs = []
    for i in range(len(nums)):
        for j in range(i+1, len(nums)):
            if nums[i] + nums[j] == target:
                pairs.append((nums[i], nums[j]))
    return pairs

# Good: O(n) - use hash set
def find_pairs_fast(nums, target):
    seen = set()
    pairs = []
    for num in nums:
        complement = target - num
        if complement in seen:
            pairs.append((complement, num))
        seen.add(num)
    return pairs
```

## Precompute Results

```python
# Bad: Recompute each time
for item in items:
    result = expensive_function(item.category)
    process(item, result)

# Good: Cache results
cache = {}
for item in items:
    if item.category not in cache:
        cache[item.category] = expensive_function(item.category)
    process(item, cache[item.category])
```

## Early Exit

```python
# Bad: Always process all
def has_negative(nums):
    count = sum(1 for n in nums if n < 0)
    return count > 0

# Good: Exit early
def has_negative(nums):
    return any(n < 0 for n in nums)
```

## Use Built-in Functions

```python
# Bad: Manual implementation
total = 0
for x in nums:
    total += x

# Good: Built-in (C-speed)
total = sum(nums)

# More examples
max_val = max(nums)          # Not manual loop
sorted_nums = sorted(nums)   # Optimized Timsort
all_positive = all(n > 0 for n in nums)
```

## Code Examples

**Two-pointer technique**

```python
# Two-pointer technique for O(n) instead of O(n²)
def two_sum_sorted(nums, target):
    """Find pair summing to target in sorted array."""
    left, right = 0, len(nums) - 1
    
    while left < right:
        current_sum = nums[left] + nums[right]
        
        if current_sum == target:
            return (left, right)
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    
    return None

# Works in O(n) for sorted input
nums = [1, 2, 3, 4, 5, 6, 7, 8, 9]
print(two_sum_sorted(nums, 10))  # (0, 8) or (1, 7), etc.
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*