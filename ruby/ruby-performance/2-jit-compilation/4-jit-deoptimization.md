---
source_course: "ruby-performance"
source_lesson: "ruby-performance-jit-deoptimization"
---

# Understanding Deoptimization

## Introduction
The JIT makes optimistic assumptions to generate fast code: it assumes types stay stable, methods are not redefined, and no debugging hooks are active. When any assumption breaks, the JIT must deoptimize—falling back to the interpreter for that code path. Understanding deoptimization helps you avoid accidentally destroying JIT performance.

## Key Concepts
- **Side exit**: A point where compiled code bails out to the interpreter because it encountered a type or condition it was not compiled for.
- **Invalidation**: When the JIT discards compiled code entirely because a global assumption (like a method definition) has changed.
- **TracePoint**: Ruby's debugging hook mechanism, which can force the JIT to invalidate all compiled code when activated.

## Real World Context
A single `TracePoint.new(:call).enable` in a debugging gem can invalidate every piece of compiled code in your process, dropping throughput back to interpreter levels. In production, ensure no debugging hooks are accidentally left active.

## Deep Dive

### Side Exits

When compiled code encounters an unexpected type, it takes a side exit back to the interpreter:

```ruby
def process(x)
  x.to_s  # Compiled assuming x is Integer
end

process(1)      # Uses compiled code — Integer#to_s
process("hi")   # Side exit! Type mismatch, falls back to interpreter
```

The first call causes YJIT to compile `process` with Integer-specialized code. The second call finds a String instead, triggering a side exit. YJIT may then recompile with a polymorphic check, but each side exit has a cost.

### Causes of Deoptimization

Four main categories trigger deoptimization:

```ruby
# 1. Type changes — different argument types than compiled for
def calculate(val)
  val * 2
end
calculate(10)    # Compiled for Integer
calculate(3.14)  # Side exit: Float

# 2. Method redefinition — class reopened, method changed
class String
  def length
    42  # Redefining a core method invalidates compiled code
  end
end

# 3. TracePoint activation — debugging hooks
TracePoint.new(:call) { |tp| puts tp.method_id }.enable
# Invalidates ALL compiled code!

# 4. Constant mutation — changing a constant's value
MAX = 100
MAX = 200  # Warning + invalidation of code using MAX
```

Each of these breaks an assumption the JIT relied on when generating machine code.

### Monitoring Deoptimization

YJIT provides statistics to track side exits and invalidations:

```ruby
stats = RubyVM::YJIT.runtime_stats

# Total invalidations from TracePoint or method redefinition
puts stats[:invalidate_everything]

# For detailed exit tracing, run with:
# ruby --yjit --yjit-trace-exits=100 script.rb
# This records the first 100 side exits with backtraces
```

The `--yjit-trace-exits=N` flag is the most actionable diagnostic. It shows you exactly which code locations cause exits, so you can fix the highest-impact ones first.

### Freezing and Constants

Freezing objects and using proper constants helps the JIT maintain its assumptions:

```ruby
# Frozen constants are safe for JIT optimization
DEFAULTS = { timeout: 30, retries: 3 }.freeze

# The JIT can inline constant lookups when values are stable
class Config
  TIMEOUT = 30  # Never reassign this
end
```

When constants are never mutated, the JIT can treat them as compile-time values.

## Common Pitfalls
1. **Leaving TracePoint enabled in production** — Even a single active TracePoint with `:call` event invalidates all compiled code. Ensure debugging gems like `binding.pry` are not loaded in production.
2. **Redefining core methods for monkey-patching** — Reopening `String`, `Array`, or `Hash` to override methods triggers global invalidation. Use Refinements instead.

## Best Practices
1. **Freeze all constants** — Freezing prevents accidental mutation and lets the JIT treat constant values as stable.
2. **Profile with --yjit-trace-exits before optimizing** — Fixing one megamorphic call site that causes 50% of side exits is more impactful than rewriting ten methods.

## Summary
- Side exits occur when compiled code encounters unexpected types, falling back to the interpreter.
- Method redefinition, TracePoint, and constant mutation cause global code invalidation.
- Use `--yjit-trace-exits=N` to find the highest-impact deoptimization sites.
- Freeze constants and avoid monkey-patching core classes to maintain JIT assumptions.

## Code Examples

**Checking for global YJIT invalidations that indicate TracePoint activation or method redefinition**

```ruby
# Monitor YJIT invalidations in a production health check
stats = RubyVM::YJIT.runtime_stats

if stats[:invalidate_everything] > 0
  warn "WARNING: #{stats[:invalidate_everything]} global invalidations detected"
  warn "Check for active TracePoints or method redefinitions"
end

puts "Code GCs: #{stats[:code_gc_count]}"
```


## Resources

- [YJIT Documentation](https://docs.ruby-lang.org/en/4.0/jit/yjit_md.html) — Official YJIT documentation covering side exits and invalidation

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*