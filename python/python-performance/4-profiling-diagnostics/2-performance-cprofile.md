---
source_course: "python-performance"
source_lesson: "python-performance-cprofile"
---

# cProfile: The Standard Profiler

## Command Line Usage

```bash
# Basic profiling
python -m cProfile script.py

# Sort by cumulative time
python -m cProfile -s cumtime script.py

# Save to file
python -m cProfile -o profile.stats script.py
```

## Programmatic Usage

```python
import cProfile
import pstats

def main():
    # Your code here
    pass

# Method 1: Context manager
with cProfile.Profile() as pr:
    main()

stats = pstats.Stats(pr)
stats.sort_stats('cumulative')
stats.print_stats(10)  # Top 10
```

## Understanding Output

```
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
      100    0.500    0.005    2.000    0.020 module.py:10(slow_func)
```

- **ncalls**: Number of calls
- **tottime**: Time in this function (excluding subcalls)
- **percall**: tottime / ncalls
- **cumtime**: Time in this function (including subcalls)
- **percall**: cumtime / ncalls

## Filtering Results

```python
stats = pstats.Stats(pr)

# Sort options: 'cumulative', 'time', 'calls'
stats.sort_stats('cumulative')

# Filter by pattern
stats.print_stats('mymodule')  # Only mymodule functions
stats.print_stats(10)  # Top 10
stats.print_stats(0.1)  # Top 10%

# Show callers/callees
stats.print_callers('slow_func')
stats.print_callees('main')
```

## Code Examples

**Profile decorator**

```python
import cProfile
import pstats
from io import StringIO

def profile_function(func):
    """Decorator to profile a function."""
    def wrapper(*args, **kwargs):
        pr = cProfile.Profile()
        pr.enable()
        result = func(*args, **kwargs)
        pr.disable()
        
        # Print stats
        s = StringIO()
        ps = pstats.Stats(pr, stream=s).sort_stats('cumulative')
        ps.print_stats(10)
        print(s.getvalue())
        
        return result
    return wrapper

@profile_function
def my_function():
    return sum(i**2 for i in range(100000))
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*