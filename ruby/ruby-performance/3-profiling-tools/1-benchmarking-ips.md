---
source_course: "ruby-performance"
source_lesson: "ruby-performance-benchmarking-ips"
---

# Benchmarking with IPS

## Introduction
Standard `Benchmark` measures total elapsed time, but that single number hides statistical noise. The `benchmark-ips` gem measures iterations per second, giving you a statistically meaningful comparison between two approaches.

## Key Concepts
- **Iterations Per Second (IPS)**: How many times a block can execute in one second, averaged over the measurement window.
- **Warmup period**: A phase before measurement that lets the JIT compile hot paths and stabilize performance.
- **Comparison mode**: An output format that ranks each report entry and shows the relative slowdown factor.

## Real World Context
When you suspect one algorithm is faster than another, micro-benchmarks settle the question with data. Every Ruby performance PR at Shopify and GitHub includes benchmark-ips output to justify the change.

## Deep Dive
The simplest usage creates a block for each approach, then calls `compare!` to see which one wins.

Here we compare `Array#sort` (returns a new array) against `Array#sort!` (sorts in place):

```ruby
require 'benchmark/ips'

Benchmark.ips do |x|
  x.report("Array#sort") { [3, 1, 2].sort }
  x.report("Array#sort!") { [3, 1, 2].sort! }
  x.compare!
end
```

The output looks like this, showing iterations per second for each report and the relative difference:

```
Array#sort:   1234567.8 i/s
Array#sort!:   987654.3 i/s - 1.25x slower
```

You can also control the warmup and measurement durations to improve accuracy:

```ruby
Benchmark.ips do |x|
  x.warmup = 2   # 2 seconds of warmup (lets YJIT compile)
  x.time = 5     # 5 seconds of measurement

  x.report("baseline") { baseline_method }
  x.report("optimized") { optimized_method }

  x.compare!
  x.hold! "results.json"  # Save for cross-run comparison
end
```

The `hold!` method saves results to a JSON file so you can compare across separate Ruby versions or commits.

## Common Pitfalls
1. **Benchmarking I/O-bound code** — IPS measures CPU throughput. If the bottleneck is a network call or disk read, the numbers are meaningless. Use wall-clock profiling instead.
2. **Forgetting warmup with YJIT** — Without a warmup period, the first iterations run interpreted code while YJIT compiles. This skews results lower than real-world performance.

## Best Practices
1. **Always use `compare!`** — Raw numbers are hard to interpret. The relative comparison ("1.3x slower") is what matters for decision-making.
2. **Benchmark on production-like hardware** — Laptop results can differ from server results due to thermal throttling, different CPU caches, and power management.

## Summary
- `benchmark-ips` measures iterations per second, which is more meaningful than total elapsed time.
- Always include a warmup period (at least 2 seconds) to let YJIT stabilize.
- Use `compare!` for side-by-side ranking and `hold!` for cross-run comparisons.

## Code Examples

**Comparing flat_map vs map+flatten — flat_map avoids allocating the intermediate array, so it wins.**

```ruby
require 'benchmark/ips'

Benchmark.ips do |x|
  x.warmup = 2
  x.time   = 5

  x.report("map + flatten") { [[1,2],[3,4]].map { |a| a }.flatten }
  x.report("flat_map")      { [[1,2],[3,4]].flat_map { |a| a } }

  x.compare!
end
# Output:
# flat_map:    3456789.0 i/s
# map+flatten: 1234567.0 i/s - 2.80x slower
```


## Resources

- [benchmark-ips on RubyGems](https://rubygems.org/gems/benchmark-ips) — Official gem page with usage examples and API documentation

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*