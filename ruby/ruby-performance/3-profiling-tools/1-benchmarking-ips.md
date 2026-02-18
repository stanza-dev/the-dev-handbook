---
source_course: "ruby-performance"
source_lesson: "ruby-performance-benchmarking-ips"
---

# Why Iterations Per Second?

Standard `Benchmark` measures total time, but `benchmark-ips` measures **iterations per second**, which is more statistically meaningful for micro-benchmarks.

## Basic Usage

```ruby
require 'benchmark/ips'

Benchmark.ips do |x|
  x.report("Array#sort") { [3, 1, 2].sort }
  x.report("Array#sort!") { [3, 1, 2].sort! }

  # Compare results
  x.compare!
end
```

## Output Interpretation

```
Array#sort:   1234567.8 i/s
Array#sort!:   987654.3 i/s - 1.25x slower
```

## Warmup Period

```ruby
Benchmark.ips do |x|
  x.warmup = 2        # 2 seconds warmup
  x.time = 5          # 5 seconds measurement

  x.report("test") { work }
end
```

## Comparing with Baseline

```ruby
Benchmark.ips do |x|
  x.report("baseline") { baseline_method }
  x.report("optimized") { optimized_method }

  x.compare!
  x.hold! "results.json"  # Save for later comparison
end
```

## When NOT to Use Micro-benchmarks

- I/O bound operations (network, disk)
- Operations with high variance
- Testing real application performance (use profiler instead)

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*