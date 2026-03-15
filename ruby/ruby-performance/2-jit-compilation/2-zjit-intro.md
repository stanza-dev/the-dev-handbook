---
source_course: "ruby-performance"
source_lesson: "ruby-performance-zjit-intro"
---

# ZJIT: Next-Generation JIT (Ruby 4.0)

## Introduction
ZJIT is a new experimental JIT compiler introduced in Ruby 4.0, designed from the ground up with a method-based compilation approach and an SSA-based intermediate representation. It is not yet production-ready but represents the future direction of Ruby's JIT infrastructure.

## Key Concepts
- **Method-based compilation**: ZJIT compiles entire methods at once, unlike YJIT which compiles individual basic blocks lazily.
- **SSA-based IR**: Static Single Assignment form, an intermediate representation where each variable is assigned exactly once, enabling powerful optimizations.
- **Experimental status**: ZJIT is included in Ruby 4.0 for testing and contributor experimentation but is not recommended for production use.

## Real World Context
ZJIT is Ruby's long-term bet on JIT performance. While YJIT delivers strong results today, ZJIT's method-level compilation and SSA IR are designed to enable optimizations (like loop-invariant code motion and global value numbering) that basic-block JITs cannot perform. Ruby core targets YJIT-level performance from ZJIT by Ruby 4.1.

## Deep Dive

### YJIT vs ZJIT Architecture

The fundamental difference is the unit of compilation:

| Aspect | YJIT | ZJIT |
|--------|------|------|
| Compilation unit | Basic block | Entire method |
| IR design | Custom | SSA-based |
| Maturity | Production-ready | Experimental |
| Current performance | Faster | Catching up |
| Extensibility | Complex internals | Designed for contributors |

YJIT compiles one basic block at a time as execution flows through it. ZJIT compiles the entire method, which gives it a wider view for optimization but requires more upfront analysis.

### Enabling ZJIT

ZJIT can be activated through multiple mechanisms:

```bash
# Via command-line flag
ruby --zjit your_script.rb

# Via environment variable
export RUBY_ZJIT_ENABLE=1
ruby your_script.rb
```

Both approaches work the same way. You can also enable ZJIT at runtime from within your Ruby program:

```ruby
# Enable at runtime
RubyVM::ZJIT.enable
```

Note that you cannot enable both YJIT and ZJIT simultaneously—they are mutually exclusive.

### Build Requirements

ZJIT is partially implemented in Rust and requires **Rust 1.85.0 or later** to compile. If you build Ruby from source without a compatible Rust toolchain, ZJIT support will be omitted:

```bash
# Ensure Rust 1.85.0+ is installed before building Ruby
rustc --version  # Must be >= 1.85.0

# Configure Ruby with ZJIT support
./configure --enable-zjit
make
```

Pre-built Ruby packages from major distributions typically include ZJIT if they include Rust in their build environment.

### Current Status in Ruby 4.0

ZJIT can compile and optimize several constructs, but it is not comprehensive yet:

```ruby
# What ZJIT handles well today:
# - Method sends (monomorphic preferred)
# - Instance variable reads and writes
# - Object allocations
# - Struct accesses
# - Some string operations
# - Optional parameters

# What it falls back to the interpreter for:
# - Complex metaprogramming (method_missing, define_method)
# - Heavily polymorphic call sites
# - Features not yet implemented in the IR
```

ZJIT is faster than the interpreter for the constructs it handles, but it does not yet match YJIT's performance across the full benchmark suite. The Ruby core team targets reaching YJIT-level performance by Ruby 4.1.

## Common Pitfalls
1. **Using ZJIT in production** — ZJIT is explicitly experimental in Ruby 4.0. It may produce incorrect results for edge cases and lacks the battle-testing YJIT has received.
2. **Expecting ZJIT to replace YJIT** — ZJIT aims to eventually match and surpass YJIT, but both compilers will coexist. YJIT remains the recommended JIT for production.

## Best Practices
1. **Experiment with ZJIT in test environments** — Run your test suite under `--zjit` to help the Ruby core team find bugs and missing optimizations.
2. **Follow ZJIT development** — As an SSA-based compiler, ZJIT will gain optimizations rapidly. Check the Ruby changelog with each release.

## Summary
- ZJIT is an experimental method-based JIT with an SSA IR, new in Ruby 4.0.
- Enable with `ruby --zjit`, `RUBY_ZJIT_ENABLE=1`, or `RubyVM::ZJIT.enable`.
- Requires Rust 1.85.0+ to build.
- Not production-ready; targets YJIT-level performance by Ruby 4.1.
- Cannot run simultaneously with YJIT.

## Code Examples

**Detecting which JIT compiler is active at runtime to log or adjust behavior**

```ruby
# Check which JIT is active
if defined?(RubyVM::ZJIT) && RubyVM::ZJIT.enabled?
  puts "ZJIT is active (experimental)"
elsif defined?(RubyVM::YJIT) && RubyVM::YJIT.enabled?
  puts "YJIT is active (production-ready)"
else
  puts "No JIT active — running interpreter only"
end
```


## Resources

- [ZJIT Documentation](https://docs.ruby-lang.org/en/4.0/jit/zjit_md.html) — Official ZJIT documentation

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*