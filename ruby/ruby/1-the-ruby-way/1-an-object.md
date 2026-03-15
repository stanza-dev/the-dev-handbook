---
source_course: "ruby"
source_lesson: "ruby-everything-is-an-object"
---

# Everything is an Object

## Introduction

Ruby is a purely object-oriented language where every single value is an object. Unlike languages such as Java or C that have primitive types, Ruby treats integers, booleans, and even `nil` as full-fledged objects with methods. Understanding this principle is the foundation for writing idiomatic Ruby code.

## Key Concepts

- **Object**: Every value in Ruby, including numbers, strings, booleans, and nil, is an instance of a class with methods you can call.
- **Class hierarchy**: All objects inherit from `BasicObject` through `Object` and `Kernel`, forming a unified type system.
- **NilClass**: Even the absence of a value (`nil`) is represented as an object of class `NilClass`.

## Real World Context

When building a web application in Rails, you frequently call methods on values that would be primitives in other languages. For example, formatting a price with `29.99.round(1)` or checking `user_count.zero?` in a conditional. Because everything is an object, you never need wrapper classes or utility functions for basic operations.

## Deep Dive

In Ruby, there are no primitives. Every value is an instance of a class. `1` is an instance of `Integer`. `true` is an instance of `TrueClass`. Even `nil` is an object!

You can verify this by calling `.class` on any value:

```ruby
1.class          # => Integer
true.class       # => TrueClass
nil.class        # => NilClass
"hello".class    # => String
```

Each of these returns the class that the value belongs to, proving that every literal is a real object.

### Calling Methods on Literals

Because everything is an object, you can call methods directly on literals without any wrapping or conversion:

```ruby
1.methods.count  # => 150+ methods available on an Integer
-1.abs           # => 1
42.even?         # => true
3.14.round       # => 3
```

This is powerful because it means numeric operations, string manipulations, and boolean checks all use the same consistent method-call syntax.

### The `nil` Object

Even `nil` is an object (of class `NilClass`). It is the only \"falsy\" value besides `false`.

```ruby
nil.nil?   # => true
nil.to_s   # => \"\"
nil.to_a   # => []
0.nil?     # => false (0 is truthy in Ruby!)
```

Notice that `nil` responds to conversion methods like `to_s` and `to_a`, returning sensible defaults. This makes nil safe to use in many contexts without explicit nil-checking.

### The Object Hierarchy

All objects ultimately inherit from `BasicObject`, through `Object` and `Kernel`:

```ruby
Integer.ancestors
# => [Integer, Numeric, Comparable, Object, Kernel, BasicObject]
```

This ancestor chain shows that `Integer` inherits numeric behavior from `Numeric`, comparison capabilities from `Comparable`, and core object behavior from `Object` and `Kernel`.

## Common Pitfalls

1. **Assuming 0 is falsy** — Coming from JavaScript or Python, developers expect `0` and `""` to be falsy. In Ruby, only `false` and `nil` are falsy. Always use explicit comparisons like `count.zero?` instead of relying on truthiness.
2. **Confusing `object_id` equality with value equality** — Two string literals with the same content may have different `object_id` values, while symbols with the same name always share an `object_id`. Use `==` for value comparison, not `equal?`.

## Best Practices

1. **Use predicate methods on objects** — Instead of `if x == 0`, prefer `if x.zero?`. Ruby objects provide expressive query methods that make code more readable.
2. **Leverage method introspection** — Use `.methods`, `.respond_to?`, and `.class` to explore objects in IRB. This is one of the fastest ways to learn new APIs.

## Summary

- Every value in Ruby is an object, including integers, booleans, and nil.
- You can call methods directly on any literal, making Ruby's syntax consistent and expressive.
- The class hierarchy rooted at `BasicObject` unifies all types under a single object model.

## Code Examples

**Demonstrates that every Ruby value is an object by calling .class and other methods on literals**

```ruby
# Every value is an object in Ruby
1.class          # => Integer
true.class       # => TrueClass
nil.class        # => NilClass
"hello".class    # => String

# Call methods directly on literals
42.even?         # => true
-1.abs           # => 1
nil.to_a         # => []
```


## Resources

- [Object Class](https://docs.ruby-lang.org/en/4.0/Object.html) — The root of the Ruby class hierarchy.

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*