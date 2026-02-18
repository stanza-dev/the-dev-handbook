---
source_course: "ruby-performance"
source_lesson: "ruby-performance-jit-optimization-strategies"
---

# Optimizing for JIT

Both YJIT and ZJIT rely on **type stability** for optimal performance.

## Type Stability (Monomorphism)

When a method always receives the same types, the JIT can inline aggressively:

```ruby
# BAD for JIT - polymorphic
def add(a, b)
  a + b
end
add(1, 2)        # Integer + Integer
add("a", "b")    # String + String
add([1], [2])    # Array + Array
# JIT must handle all cases!

# GOOD for JIT - monomorphic
def add_integers(a, b)
  a + b
end
# Always receives integers, JIT can optimize fully
```

## Polymorphic Inline Caches

The JIT uses inline caches to remember types. When types change, it must "deoptimize":

```ruby
# Method call site remembers receiver type
objects.each { |obj| obj.process }  
# If all objects are same class -> fast
# If mixed classes -> slower (megamorphic)
```

## Tips for JIT Performance

1. **Keep methods type-stable** - same argument types
2. **Avoid excessive metaprogramming** - `method_missing` is slow
3. **Prefer instance variables** - faster than method calls
4. **Initialize all ivars in constructor** - helps Object Shapes
5. **Use specific methods** - `each` over `map` if you don't need result

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*