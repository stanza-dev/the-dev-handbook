---
source_course: "ruby-performance"
source_lesson: "ruby-performance-stackprof"
---

# StackProf: Sampling Profiler

StackProf is a sampling CPU profiler - it periodically records what code is executing.

## Profiling Modes

```ruby
require 'stackprof'

# CPU mode - actual CPU time
StackProf.run(mode: :cpu, out: 'cpu.dump') do
  expensive_operation
end

# Wall mode - includes I/O waiting time
StackProf.run(mode: :wall, out: 'wall.dump') do
  operation_with_io
end

# Object mode - track allocations
StackProf.run(mode: :object, out: 'obj.dump') do
  allocation_heavy_code
end
```

## Analyzing Results

```bash
# Text report
stackprof cpu.dump --text

# Method breakdown
stackprof cpu.dump --method 'MyClass#process'

# Generate flamegraph data
stackprof cpu.dump --flamegraph > flamegraph.js
```

## Understanding Output

```
==== TOTAL (1000 samples)
      TOTAL    (pct)     SAMPLES    (pct)     FRAME
        800  (80.0%)         500  (50.0%)     MyClass#slow_method
        300  (30.0%)         200  (20.0%)     Array#map
```

- **TOTAL**: Time spent in method + callees
- **SAMPLES**: Time spent in method only (self time)

## Inline Profiling

```ruby
StackProf.start(mode: :cpu)
# ... your code ...
StackProf.stop
results = StackProf.results
```

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*