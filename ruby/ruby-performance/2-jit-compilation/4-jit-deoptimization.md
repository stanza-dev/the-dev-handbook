---
source_course: "ruby-performance"
source_lesson: "ruby-performance-jit-deoptimization"
---

# When JIT Falls Back

The JIT makes assumptions to generate fast code. When assumptions break, it must "deoptimize."

## Side Exits

When compiled code encounters an unexpected situation:

```ruby
def process(x)
  x.to_s  # Compiled assuming x is Integer
end

process(1)      # Uses compiled code
process("hi")   # Side exit! Falls back to interpreter
```

## Causes of Deoptimization

1. **Type changes** - Different argument types than expected
2. **Method redefinition** - Class reopened, method changed
3. **TracePoint activation** - Debugging hooks invalidate code
4. **Constant mutation** - Changing constants

## TracePoint Invalidation

```ruby
# This invalidates ALL compiled code!
TracePoint.new(:call) { |tp| puts tp.method_id }.enable

# Ruby 4.0 tracks this:
RubyVM::YJIT.runtime_stats[:invalidate_everything]
```

## Monitoring Deoptimization

```ruby
# With YJIT stats build
stats = RubyVM::YJIT.runtime_stats

# Look for high side_exit_count
puts stats[:side_exit_count]
puts stats[:total_exits]
```

## Best Practices

- **Don't redefine methods** in production
- **Avoid TracePoint** in hot paths
- **Freeze constants** to prevent accidental mutation
- **Profile first** - not everything needs JIT optimization

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*