---
source_course: "ruby-performance"
source_lesson: "ruby-performance-production-monitoring"
---

# Measuring & Monitoring in Production

## Introduction
Optimizing Ruby performance without measurement is guesswork. Production environments behave differently from development: real traffic patterns, concurrent requests, and long-running processes reveal problems that benchmarks miss. This lesson covers the essential tools and metrics for monitoring Ruby performance in production, from GC statistics to YJIT telemetry to continuous profiling.

## Key Concepts
- **`GC.stat`**: A hash of garbage collector statistics including heap size, object counts, and GC timing. The primary tool for understanding memory behavior at runtime.
- **YJIT Stats**: Runtime statistics from Ruby's JIT compiler showing compilation rates, exit reasons, and code size — available in production with minimal overhead.
- **Memory Growth Detection**: Identifying whether a process's memory is growing linearly (leak) or plateauing (normal warmup) over time.
- **Continuous Profiling**: Sampling a production process at low frequency to build flamegraphs without measurable performance impact.

## Real World Context
A Rails application running on Puma with 4 workers might show stable memory during the first hour, then one worker starts growing by 10MB per hour. Without monitoring `GC.stat` and RSS over time, this leak would go undetected until the process is OOM-killed, causing a brief outage. Production monitoring turns these silent failures into actionable alerts.

## Deep Dive

### Essential `GC.stat` Metrics

`GC.stat` returns a hash with dozens of keys. These are the ones that matter most in production:

```ruby
stats = GC.stat

# Heap usage
stats[:heap_live_slots]       # Objects currently alive
stats[:heap_free_slots]       # Empty slots available
stats[:heap_allocated_pages]  # Total pages allocated by Ruby heap

# GC frequency
stats[:minor_gc_count]        # Fast young-generation collections
stats[:major_gc_count]        # Slow full collections
stats[:total_allocated_objects]  # Lifetime object count

# Time spent in GC (Ruby 3.1+)
stats[:time]                  # Time spent in GC (platform-dependent units)
```

The ratio of `major_gc_count` to `minor_gc_count` tells you whether objects are surviving too long. A healthy ratio is roughly 1:20 or higher. If major GCs are frequent, objects are being promoted to the old generation unnecessarily.

You can expose these metrics via a health endpoint:

```ruby
# config/routes.rb (Rails)
get '/health/gc', to: proc {
  [200, { 'Content-Type' => 'application/json' }, [GC.stat.to_json]]
}
```

### YJIT Stats in Production

YJIT can report compilation statistics at runtime with very low overhead:

```ruby
# Enable YJIT stats collection (add to boot)
RubyVM::YJIT.enable  # Minimal overhead in Ruby 4.0

# Later, inspect stats
stats = RubyVM::YJIT.runtime_stats
puts stats[:inline_code_size]      # Memory used by compiled code
puts stats[:compiled_iseq_count]  # Number of methods compiled
puts stats[:ratio_in_yjit]          # % of execution in JIT code
puts stats[:side_exit_count]       # Number of times JIT exited to interpreter
```

The `ratio_in_yjit` metric is the single most important number. A healthy production Rails app should show 85-95% of execution in YJIT-compiled code. If this number is low, look at `side_exit_count` to find methods that YJIT cannot optimize.

### Detecting Memory Growth

Memory leaks in Ruby are often slow — 1-5 MB per hour. Detect them by tracking RSS over time:

```ruby
# Track RSS at regular intervals
def current_rss_mb
  `ps -o rss= -p #{Process.pid}`.to_i / 1024.0
end

# Log every 5 minutes (in a background thread or periodic job)
Thread.new do
  loop do
    rss = current_rss_mb
    gc = GC.stat
    Rails.logger.info(
      "memory_monitor rss=#{rss}MB " \
      "live_slots=#{gc[:heap_live_slots]} " \
      "old_objects=#{gc[:old_objects]} " \
      "major_gc=#{gc[:major_gc_count]}"
    )
    sleep 300
  end
end
```

If RSS grows linearly while `heap_live_slots` is stable, the leak is in native memory (C extensions, IO buffers). If `heap_live_slots` grows, the leak is in Ruby objects — use `ObjectSpace` to find the culprit.

### Vernier: Low-Overhead Production Profiling

Vernier is a sampling profiler designed for production use. It uses Ruby's signal-based sampling to build flamegraphs with less than 2% overhead:

```ruby
# Gemfile
gem 'vernier'

# Profile a specific code path in production
Vernier.trace(output: "/tmp/profile.json") do
  # The code you want to profile
  HeavyReport.generate(params)
end

# Or profile for a fixed duration
Vernier.trace(output: "/tmp/profile.json", duration: 30) do
  # Profiles the next 30 seconds of execution
  sleep 30
end
```

The output is a JSON file viewable in the Firefox Profiler or speedscope.app. Look for tall stacks (deep call chains) and wide bars (hot methods).

### derailed_benchmarks for Memory Baselines

The `derailed_benchmarks` gem measures memory consumption at boot and per-request, establishing baselines for regression detection:

```bash
# Measure memory used at boot
bundle exec derailed bundle:mem

# Measure memory growth per request
bundle exec derailed exec perf:mem_over_time

# Find which gems use the most memory
bundle exec derailed bundle:objects
```

Run these in CI to catch memory regressions before deployment. A spike in `bundle:mem` means a gem update increased baseline memory.

## Common Pitfalls
1. **Monitoring RSS without GC.stat** — RSS alone does not tell you whether growth is in Ruby objects or native memory. Always correlate RSS with `heap_live_slots` and `old_objects` to distinguish Ruby-level leaks from C-extension leaks.
2. **Enabling full YJIT stats in high-traffic production** — While stats overhead is low in Ruby 4.0, collecting detailed exit information (`stats: true` with full counters) on every process may add up. Enable on one worker and rotate.
3. **Profiling too short a window** — Memory leaks are often slow. Profile for at least 30 minutes to see trends, not snapshots. A 5-second Vernier trace will not catch a leak growing at 2MB/hour.

## Best Practices
1. **Expose GC.stat as a Prometheus endpoint** — Export `heap_live_slots`, `major_gc_count`, `old_objects`, and GC time as gauge metrics. Set alerts when major GC count spikes or old objects grow monotonically.
2. **Run `derailed bundle:mem` in CI** — Fail the build if baseline memory exceeds a threshold. This catches gem bloat and accidental require-time allocations before they reach production.
3. **Use Vernier for targeted investigation** — Do not run continuous profiling on every request. Instead, trigger Vernier traces on demand (via admin endpoint or feature flag) when you suspect a performance regression.

## Summary
- `GC.stat` provides the essential metrics: `heap_live_slots`, `major_gc_count`, `old_objects`, and GC `time`.
- YJIT's `ratio_in_yjit` tells you what percentage of execution is JIT-compiled; aim for 85%+.
- Track RSS alongside `heap_live_slots` to distinguish Ruby-level leaks from native memory growth.
- Vernier provides production-safe flamegraph profiling with under 2% overhead.
- `derailed_benchmarks` establishes memory baselines for CI-based regression detection.

## Code Examples

**A health endpoint combining GC stats, RSS tracking, and YJIT metrics — suitable for Prometheus scraping or manual inspection**

```ruby
# Production health endpoint exposing key performance metrics
class HealthController < ApplicationController
  def performance
    gc = GC.stat
    yjit = defined?(RubyVM::YJIT) ? RubyVM::YJIT.runtime_stats : {}

    render json: {
      rss_mb: `ps -o rss= -p #{Process.pid}`.to_i / 1024.0,
      gc: {
        live_slots: gc[:heap_live_slots],
        old_objects: gc[:old_objects],
        major_gc_count: gc[:major_gc_count],
        gc_time_ms: gc[:time]
      },
      yjit: {
        ratio_in_yjit: yjit[:ratio_in_yjit],
        compiled_methods: yjit[:compiled_iseq_count],
        code_size_bytes: yjit[:yjit_alloc_size]
      }
    }
  end
end
```


## Resources

- [GC Module Documentation](https://docs.ruby-lang.org/en/4.0/GC.html) — Official Ruby documentation for GC.stat and garbage collection tuning
- [Vernier Profiler](https://github.com/jhawthorn/vernier) — Low-overhead sampling profiler for Ruby, designed for production use

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*