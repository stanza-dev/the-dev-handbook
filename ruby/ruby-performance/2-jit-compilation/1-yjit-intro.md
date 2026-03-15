---
source_course: "ruby-performance"
source_lesson: "ruby-performance-yjit-intro"
---

# YJIT: Production-Ready JIT

## Introduction
YJIT (Yet Another JIT) was introduced in Ruby 3.1 and became production-ready in 3.2. It compiles frequently-called Ruby bytecode into native machine code at runtime, delivering significant speedups for real-world applications. Ruby 4.0 refines its CLI options and statistics APIs.

## Key Concepts
- **YJIT**: A basic-block-based JIT compiler that lazily compiles Ruby methods as they are called.
- **Call threshold**: The number of times a method must be called before YJIT compiles it (default: 30).
- **Code GC**: YJIT's mechanism for reclaiming compiled code that is no longer hot, freeing memory.
- **Side exit**: When compiled code encounters an unexpected type or condition and falls back to the interpreter.

## Real World Context
Shopify runs YJIT on all production Ruby services. Typical Rails applications see 15–30% throughput improvements with no code changes—just enabling the flag.

## Deep Dive

### Enabling YJIT

You can enable YJIT in several ways:

```bash
# Via command-line flag
ruby --yjit your_script.rb

# Via environment variable
export RUBY_YJIT_ENABLE=1
ruby your_script.rb
```

Both approaches activate YJIT before your code runs. The environment variable is convenient for deployment configurations where you cannot change the ruby invocation.

### Runtime Activation and Options

You can also enable YJIT at runtime, which is useful for conditional activation:

```ruby
# Enable at runtime with configuration
RubyVM::YJIT.enable

# Check if YJIT is active
RubyVM::YJIT.enabled?  # => true
```

Ruby 4.0 introduced the `--yjit-mem-size=<value>` CLI flag for setting the memory limit in megabytes. This replaces older memory configuration approaches:

```bash
# Set YJIT memory limit to 256 MB
ruby --yjit --yjit-mem-size=256 your_script.rb
```

The default memory budget is 128 MB, which is sufficient for most applications.

### YJIT Debugging and Profiling Options

Ruby 4.0 provides several diagnostic flags for understanding YJIT behavior:

```bash
# Trace the first N exits to understand deoptimization
ruby --yjit --yjit-trace-exits=50 script.rb

# Generate perf-compatible codegen annotations
ruby --yjit --yjit-perf=codegen script.rb

# Enable YJIT event logging
ruby --yjit --yjit-log script.rb
```

`--yjit-trace-exits=N` records the first N side exits with backtraces, helping you find where YJIT deoptimizes. `--yjit-perf=codegen` emits annotations for Linux perf profiling. `--yjit-log` outputs internal YJIT events for debugging.

### YJIT Statistics

Runtime stats are always available in Ruby 4.0. However, `ratio_in_yjit` requires a special build:

```ruby
# Always available in Ruby 4.0
stats = RubyVM::YJIT.runtime_stats
puts stats[:inline_code_size]       # Compiled code bytes
puts stats[:code_gc_count]          # Code GC runs
puts stats[:invalidate_everything]  # TracePoint invalidations

# ratio_in_yjit requires building Ruby with:
#   --enable-yjit=stats
# Without that build flag, ratio_in_yjit is not reported.
```

Monitoring `inline_code_size` tells you how much memory YJIT is using for compiled code. If `code_gc_count` is high, consider increasing `--yjit-mem-size`.

## Common Pitfalls
1. **Expecting ratio_in_yjit without a stats build** — The `ratio_in_yjit` metric is only available when Ruby is compiled with `--enable-yjit=stats`. Standard release builds omit it for performance.
2. **Setting mem-size too low** — If YJIT runs out of code memory, it triggers code GC frequently, which can negate performance gains. Monitor `code_gc_count`.

## Best Practices
1. **Enable YJIT in production** — It is stable and provides free throughput gains for most Ruby workloads with no code changes required.
2. **Use --yjit-trace-exits during optimization** — It pinpoints exactly where YJIT deoptimizes, letting you fix the hottest side-exit sites first.

## Summary
- YJIT compiles Ruby bytecode to machine code at runtime for 15–30% speedups.
- Enable via `--yjit`, `RUBY_YJIT_ENABLE=1`, or `RubyVM::YJIT.enable`.
- `--yjit-mem-size=<value>` sets the code memory budget in MB.
- `--yjit-trace-exits=N`, `--yjit-perf=codegen`, and `--yjit-log` aid debugging.
- `ratio_in_yjit` requires a `--enable-yjit=stats` build.

## Code Examples

**Enabling YJIT at runtime and reading key statistics to monitor compilation and invalidation**

```ruby
# Enable YJIT at runtime and inspect stats
RubyVM::YJIT.enable

# After running workload...
stats = RubyVM::YJIT.runtime_stats
puts "Code size:   #{stats[:inline_code_size]} bytes"
puts "Code GCs:    #{stats[:code_gc_count]}"
puts "Invalidated: #{stats[:invalidate_everything]}"
```


## Resources

- [YJIT Documentation](https://docs.ruby-lang.org/en/4.0/jit/yjit_md.html) — Official YJIT documentation

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*