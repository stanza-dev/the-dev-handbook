---
source_course: "ruby-performance"
source_lesson: "ruby-performance-jit-optimization-strategies"
---

# Writing JIT-Friendly Code

## Introduction
Both YJIT and ZJIT rely on type stability to generate efficient machine code. When a method always receives the same types, the JIT can specialize aggressively. When types vary, it must generate slower, generic code or fall back to the interpreter.

## Key Concepts
- **Monomorphism**: A call site where the receiver is always the same class, allowing the JIT to inline the method directly.
- **Polymorphism**: A call site with 2–4 different receiver types, requiring a type check before dispatching.
- **Megamorphism**: A call site with many receiver types, where the JIT gives up on specialization entirely.
- **Inline cache**: A per-call-site cache that remembers the receiver type from the previous call to speed up the next one.

## Real World Context
In a typical Rails controller, polymorphic `render` calls and dynamic `send` invocations are JIT-unfriendly hot spots. Refactoring the hottest call sites to be monomorphic can improve YJIT throughput by 5–15% in those paths.

## Deep Dive

### Type Stability

The JIT generates specialized code based on observed types. When those types are consistent, the specialized code runs every time:

```ruby
# BAD for JIT — polymorphic call site
def add(a, b)
  a + b
end
add(1, 2)        # Integer + Integer
add("a", "b")    # String + String
add([1], [2])    # Array + Array
# JIT must handle all three cases!
```

The `+` method resolves to three different implementations. The JIT cannot inline any of them safely because it does not know which will be called next.

```ruby
# GOOD for JIT — monomorphic
def add_integers(a, b)
  a + b
end
# Always receives Integers, so JIT compiles Integer#+ directly
```

Separating by type lets the JIT generate a single specialized path.

### Inline Caches and Megamorphism

Ruby uses inline caches at each call site to remember the receiver type:

```ruby
# Each call site has its own inline cache
objects.each { |obj| obj.process }
# If all objects are the same class -> monomorphic (fast)
# If 2-4 classes -> polymorphic (slower, type checks)
# If many classes -> megamorphic (no caching, slow)
```

When a cache goes megamorphic, the JIT cannot specialize at all and every call goes through the full method lookup.

### Tips for JIT Performance

The following practices help both YJIT and ZJIT generate better code:

```ruby
# 1. Keep methods type-stable
# Same argument types at each call site

# 2. Avoid excessive metaprogramming
# method_missing and define_method defeat JIT optimization

# 3. Prefer instance variables over method calls
# @name is faster than name() under JIT

# 4. Initialize all ivars in the constructor
# Consistent Object Shapes help the JIT access ivars by index

# 5. Use specific methods over generic ones
# each_with_object over inject when accumulating
```

Each tip reduces the number of type checks and fallback paths the JIT must generate.

## Common Pitfalls
1. **Over-using method_missing** — Every call through `method_missing` is invisible to the JIT and bypasses inline caches entirely. Use it sparingly.
2. **Mixing types in collections** — An array containing Integers, Strings, and Hashes forces every `each` block to be megamorphic.

## Best Practices
1. **Profile before optimizing** — Use `--yjit-trace-exits` to find actual deoptimization sites rather than guessing. Optimize the hottest exits first.
2. **Initialize all instance variables upfront** — Consistent Object Shapes let the JIT access ivars via direct index rather than hash lookup.

## Summary
- Monomorphic call sites let the JIT inline methods; megamorphic sites force generic dispatch.
- Inline caches store the receiver type per call site and degrade under type diversity.
- Avoid `method_missing`, mixed-type collections, and dynamic `send` in hot paths.
- Profile with `--yjit-trace-exits` to find the real deoptimization bottlenecks.

## Code Examples

**A monomorphic loop that the JIT can fully specialize because all elements are Integers**

```ruby
# Monomorphic: JIT compiles Integer#+ directly
def sum_integers(arr)
  total = 0
  arr.each { |n| total += n }  # Always Integer
  total
end

result = sum_integers([1, 2, 3, 4, 5])
# => 15  (JIT-optimized, no type checks needed)
```


## Resources

- [YJIT Documentation](https://docs.ruby-lang.org/en/4.0/jit/yjit_md.html) — Official YJIT documentation including optimization tips

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*