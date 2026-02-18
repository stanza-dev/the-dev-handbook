---
source_course: "ruby-performance"
source_lesson: "ruby-performance-zjit-intro"
---

# ZJIT: Experimental in Ruby 4.0

ZJIT is a new method-based JIT compiler introduced in Ruby 4.0, partially built with Rust.

## Key Differences from YJIT

| Aspect | YJIT | ZJIT |
|--------|------|------|
| Architecture | Basic-block based | Method-based |
| Maturity | Production-ready | Experimental |
| Performance | Faster (for now) | Catching up |
| IR | Custom | SSA-based |
| Extensibility | Complex | Designed for contributors |

## Enabling ZJIT

```bash
# Command line
ruby --zjit your_script.rb

# Environment variable
export RUBY_ZJIT_ENABLE=1

# Runtime (Ruby 4.0)
RubyVM::ZJIT.enable
```

## Build Requirements

ZJIT requires Rust 1.85.0+ to build Ruby with ZJIT support.

## Current Status (Ruby 4.0)

- **Faster than interpreter**: Yes
- **Faster than YJIT**: Not yet (targeting Ruby 4.1)
- **Production use**: Not recommended yet

## What ZJIT Optimizes

- Method sends (monomorphic preferred)
- Instance variable reads/writes
- Object allocations
- Struct accesses
- Some string operations
- Optional parameters

## Resources

- [ZJIT Documentation](https://docs.ruby-lang.org/en/4.0/jit/zjit_md.html) — Official ZJIT documentation

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*