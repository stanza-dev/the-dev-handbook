---
source_course: "ruby-performance"
source_lesson: "ruby-performance-yjit-intro"
---

# YJIT (Yet Another JIT)

Introduced in Ruby 3.1 and production-ready since 3.2, YJIT compiles Ruby bytecode to machine code at runtime.

## Enabling YJIT

```bash
# Command line
ruby --yjit your_script.rb

# Environment variable
export RUBY_YJIT_ENABLE=1
ruby your_script.rb
```

## Runtime Enable (Ruby 3.2+)

```ruby
# Enable at runtime with options
RubyVM::YJIT.enable(
  mem_size: 128,        # Memory in MB (default: 128)
  call_threshold: 30    # Calls before compiling (default: 30)
)

# Check if enabled
RubyVM::YJIT.enabled?  # => true
```

## YJIT Statistics (Ruby 4.0)

```ruby
# Get runtime stats (always available in 4.0)
stats = RubyVM::YJIT.runtime_stats

puts stats[:inline_code_size]       # Compiled code size
puts stats[:code_gc_count]          # Code GC runs
puts stats[:invalidate_everything]  # TracePoint invalidations

# Note: ratio_in_yjit requires --enable-yjit=stats build
```

## Performance Impact

YJIT typically provides:
- 15-30% speedup for Rails apps
- Higher gains for CPU-intensive code
- Minimal impact on memory (uses ~128MB by default)

## Resources

- [YJIT Documentation](https://docs.ruby-lang.org/en/4.0/jit/yjit_md.html) — Official YJIT documentation

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*