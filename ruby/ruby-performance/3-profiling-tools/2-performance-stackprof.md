---
source_course: "ruby-performance"
source_lesson: "ruby-performance-stackprof"
---

# CPU Profiling with StackProf

## Introduction
Benchmarks tell you which code is faster, but not where the bottleneck lives. StackProf is a sampling CPU profiler that periodically snapshots the call stack to reveal which methods consume the most time.

## Key Concepts
- **Sampling profiler**: Instead of instrumenting every method call, StackProf interrupts the program at regular intervals and records the current stack. This keeps overhead low (typically under 5%).
- **Self time (SAMPLES)**: Time spent inside a method itself, excluding its callees.
- **Total time (TOTAL)**: Time spent in a method plus all the methods it calls.
- **Profiling mode**: StackProf supports `:cpu` (actual CPU cycles), `:wall` (elapsed clock time including I/O waits), and `:object` (counts object allocations).

## Real World Context
Every Rails performance investigation starts with a profiler. When a request takes 800ms instead of 200ms, StackProf shows you the exact method responsible. Without it, you are guessing.

## Deep Dive
StackProf wraps the code you want to profile in a block. The `mode` parameter controls what is measured.

Here we profile a CPU-intensive operation to find where compute time is spent:

```ruby
require 'stackprof'

StackProf.run(mode: :cpu, out: 'cpu.dump') do
  expensive_operation
end
```

After profiling, analyze the dump from the command line:

```bash
# Top methods by self time
stackprof cpu.dump --text

# Detailed breakdown of a specific method
stackprof cpu.dump --method 'MyClass#process'
```

The text output shows a table like this, where SAMPLES is self time and TOTAL includes callees:

```
==== TOTAL (1000 samples)
      TOTAL    (pct)     SAMPLES    (pct)     FRAME
        800  (80.0%)         500  (50.0%)     MyClass#slow_method
        300  (30.0%)         200  (20.0%)     Array#map
```

For wall-clock profiling that includes I/O waits, switch the mode:

```ruby
StackProf.run(mode: :wall, out: 'wall.dump') do
  operation_with_io
end
```

You can also start and stop profiling manually for more control:

```ruby
StackProf.start(mode: :cpu)
# ... your code ...
StackProf.stop
results = StackProf.results
```

## Common Pitfalls
1. **Using :cpu mode for I/O-heavy code** — CPU mode only samples when the CPU is active. If your bottleneck is waiting on a database, those waits are invisible. Use `:wall` mode instead.
2. **Profiling too short a window** — With fewer than 100 samples the results are statistically noisy. Run the profiled code long enough to collect at least a few hundred samples.

## Best Practices
1. **Start with :wall mode in production** — It captures the full picture including I/O, then switch to `:cpu` once you isolate compute-bound hotspots.
2. **Use `--method` for drill-down** — After identifying the top method, use `stackprof dump --method 'ClassName#method'` to see line-by-line sample counts.

## Summary
- StackProf samples the call stack at intervals to find performance hotspots.
- Use `:cpu` for computation, `:wall` for I/O-inclusive timing, `:object` for allocation tracking.
- SAMPLES (self time) tells you where the CPU is actually working; TOTAL includes callees.

## Code Examples

**Using StackProf in :object mode to find allocation hotspots — each string concatenation creates a new String object.**

```ruby
require 'stackprof'

# Object mode tracks allocations instead of CPU time
StackProf.run(mode: :object, out: 'alloc.dump') do
  1000.times { "hello" + " world" }  # each concat allocates a new String
end

# Analyze: stackprof alloc.dump --text
# Shows which methods allocate the most objects
```


## Resources

- [StackProf GitHub Repository](https://github.com/tmm1/stackprof) — Source code, usage instructions, and examples for the StackProf sampling profiler

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*