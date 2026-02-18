---
source_course: "ruby-performance"
source_lesson: "ruby-performance-flamegraphs"
---

# Flamegraphs

Flamegraphs are visualizations of profiled stack traces that make hotspots immediately obvious.

## Reading a Flamegraph

- **X-axis**: Population (how often that stack was sampled)
- **Y-axis**: Stack depth (callers below, callees above)
- **Width**: Proportional to time spent
- **Color**: Usually random or grouped by category

## Wide Bars = Hotspots

```
                    +--slow_method--+
          +--------process---------+
    +------------handle_request-----------+
+------------------main------------------------+
```

Wide bars at the TOP indicate methods using CPU time directly.

## Generating Flamegraphs

```ruby
# With StackProf
StackProf.run(mode: :cpu, raw: true, out: 'profile.dump') do
  your_code
end

# Convert to flamegraph
# stackprof profile.dump --d3-flamegraph > flame.html
```

## Using Vernier (Modern Alternative)

```ruby
require 'vernier'

Vernier.profile(out: 'profile.json') do
  your_code
end

# Open profile.json in Firefox Profiler or Speedscope
```

## Tips for Analysis

1. Look for **wide plateaus** at the top
2. Check for **unexpected depth** (too many layers)
3. Compare **before/after** optimizations
4. Filter by **specific threads** if needed

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*