---
source_course: "ruby-performance"
source_lesson: "ruby-performance-flamegraphs"
---

# Visualizing with Flamegraphs

## Introduction
A table of method names and sample counts can be hard to scan when you have hundreds of methods. Flamegraphs transform profiling data into a visual stack chart where performance bottlenecks jump out immediately.

## Key Concepts
- **Flamegraph**: A visualization of sampled stack traces where the x-axis represents the proportion of samples and the y-axis represents call depth.
- **Plateau**: A wide bar at the top of the stack, indicating a method that consumes significant CPU time directly (high self time).
- **Speedscope / Firefox Profiler**: Browser-based tools that render interactive flamegraphs from JSON profiling data.

## Real World Context
When triaging a slow endpoint, engineers share flamegraph screenshots in pull requests and incident channels. A single image tells the story faster than pages of profiler text output.

## Deep Dive
Flamegraphs encode three things visually:

- **Width** of a bar is proportional to how many samples included that method. Wider bars mean more time.
- **Vertical position** shows the call hierarchy. The bottom is the entry point (e.g., `main`), and the top holds leaf methods doing actual work.
- **Color** is typically random or grouped by module. It carries no performance meaning.

A simplified ASCII flamegraph looks like this, where `slow_method` is the hotspot:

```
                    +--slow_method--+
          +--------process---------+
    +------------handle_request-----------+
+------------------main------------------------+
```

To generate a flamegraph from StackProf, pass `raw: true` when profiling:

```ruby
require 'stackprof'

StackProf.run(mode: :cpu, raw: true, out: 'profile.dump') do
  your_code
end
```

Then convert the dump to an interactive HTML flamegraph:

```bash
stackprof profile.dump --d3-flamegraph > flame.html
```

Vernier is a modern alternative that outputs JSON compatible with Firefox Profiler and Speedscope:

```ruby
require 'vernier'

Vernier.profile(out: 'profile.json') do
  your_code
end
# Open profile.json at https://profiler.firefox.com or https://speedscope.app
```

Vernier captures thread activity and GC pauses automatically, giving richer context than StackProf alone.

## Common Pitfalls
1. **Confusing width with call count** — A wide bar means more *samples*, not necessarily more calls. A method called once but running for 500ms will be wider than a method called 10,000 times but finishing in 1ms total.
2. **Ignoring the top of the stack** — Beginners focus on the wide bars at the bottom (entry points). The actionable hotspots are the wide bars at the *top*, where the CPU actually spends time.

## Best Practices
1. **Compare before and after** — Generate flamegraphs before and after your optimization. The visual diff makes it obvious whether you shrunk the right hotspot.
2. **Use Vernier for GC visibility** — Vernier annotates GC pauses directly in the flamegraph, so you can see whether garbage collection is a significant contributor.

## Summary
- Flamegraphs turn profiler data into a visual chart where wide bars at the top indicate hotspots.
- Use `stackprof --d3-flamegraph` or Vernier with Speedscope/Firefox Profiler for interactive exploration.
- Always compare flamegraphs before and after optimization to verify your changes had the intended effect.

## Code Examples

**Profiling an ActiveRecord query with Vernier — the output can be opened in Firefox Profiler for an interactive flamegraph.**

```ruby
require 'vernier'

# Profile a block and write output for Firefox Profiler
Vernier.profile(out: 'profile.json') do
  users = User.where(active: true).includes(:posts)
  users.each { |u| u.posts.map(&:title) }
end

# Open the resulting file:
# 1. Go to https://profiler.firefox.com
# 2. Drag and drop profile.json
# 3. Look for wide bars at the top of the flamegraph
```


## Resources

- [Vernier GitHub Repository](https://github.com/jhawthorn/vernier) — Next-generation Ruby profiler with GC annotations and Firefox Profiler integration

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*