---
source_course: "ruby"
source_lesson: "ruby-procs-vs-lambdas"
---

# Procs vs Lambdas

## Introduction

Ruby has two types of closures that are both instances of the `Proc` class but behave differently: Procs and lambdas. They differ in how they handle arguments and how `return` works. Understanding these differences is crucial because choosing the wrong one can cause subtle bugs, especially with the `return` behavior.

## Key Concepts

- **Arity checking**: Lambdas enforce strict argument counts (like methods), while Procs silently ignore extra arguments and fill missing ones with `nil`.
- **Return behavior**: `return` in a lambda exits only the lambda. `return` in a Proc exits the enclosing method, which can cause unexpected control flow.
- **Arrow syntax (`->`)**: The preferred shorthand for creating lambdas, e.g., `->(x) { x * 2 }`.

## Real World Context

When building a callback system or strategy pattern, you need to choose between Procs and lambdas. Lambdas are almost always the safer choice because they behave like methods: they check argument counts and `return` only exits the lambda. Procs are mainly used in DSL implementations where flexible argument handling and flow control through the enclosing method are intentional design choices.

## Deep Dive

Both Procs and Lambdas are instances of `Proc`, but they differ in two key ways.

### 1. Argument Handling (Arity)

Lambdas are strict about arguments, just like methods:

```ruby
lam = ->(a, b) { a + b }
lam.call(1)         # ArgumentError!
lam.call(1, 2)      # => 3
lam.call(1, 2, 3)   # ArgumentError!
```

Passing too few or too many arguments raises an `ArgumentError`, making lambdas predictable and safe.

Procs are lenient with arguments:

```ruby
proc = Proc.new { |a, b| [a, b] }
proc.call(1)        # => [1, nil]
proc.call(1, 2)     # => [1, 2]
proc.call(1, 2, 3)  # => [1, 2] (extra ignored)
```

Missing arguments become `nil` and extra arguments are silently discarded. This flexibility can be useful but also masks bugs.

### 2. Return Behavior

In a lambda, `return` exits only the lambda and returns to the calling code:

```ruby
def test_lambda
  lam = -> { return 10 }
  lam.call
  puts \"After lambda\"  # This runs!
end
test_lambda # prints \"After lambda\"
```

The method continues executing after the lambda returns, which is the behavior most developers expect.

In a Proc, `return` exits the enclosing method entirely:

```ruby
def test_proc
  pr = Proc.new { return 10 }
  pr.call
  puts \"After proc\"  # Never reached!
end
test_proc  # Returns 10, no output
```

This means a `return` inside a Proc can cause the surrounding method to exit early, which is often surprising.

### Creating Lambdas

Ruby provides two syntaxes for creating lambdas:

```ruby
# Arrow syntax (preferred)
lam = ->(x) { x * 2 }

# Traditional syntax
lam = lambda { |x| x * 2 }
```

The arrow syntax (`->`) is more concise and is the preferred style in modern Ruby.

## Common Pitfalls

1. **Using `return` inside a Proc passed to an iterator** — A `return` inside a Proc used with `each` or `map` will exit the enclosing method, not just the current iteration. Use `next` to skip to the next iteration instead.
2. **Assuming Proc and lambda are interchangeable** — Because both are `Proc` instances, developers sometimes swap them without realizing the arity and return differences. Use `proc.lambda?` to check which type you have.

## Best Practices

1. **Default to lambdas for callbacks and stored callables** — Lambdas are safer because they enforce argument counts and have predictable return behavior. Use Procs only when you specifically need their flexible behavior.
2. **Use the arrow syntax for lambdas** — Write `->(x) { x * 2 }` instead of `lambda { |x| x * 2 }`. The arrow syntax is shorter and visually distinct from `Proc.new`.

## Summary

- Lambdas check argument counts strictly; Procs silently adjust by ignoring extras and filling missing ones with `nil`.
- `return` in a lambda exits the lambda; `return` in a Proc exits the enclosing method.
- Default to lambdas for most use cases; they behave like methods and are less surprising.

## Code Examples

**Contrasts lambda and Proc behavior for argument handling and type checking**

```ruby
# Lambda: strict arity, local return
lam = ->(a, b) { a + b }
lam.call(1, 2)      # => 3
lam.call(1)         # ArgumentError!

# Proc: lenient arity, non-local return
proc = Proc.new { |a, b| [a, b] }
proc.call(1)        # => [1, nil]
proc.call(1, 2, 3)  # => [1, 2]

# Check type
lam.lambda?         # => true
proc.lambda?        # => false
```


## Resources

- [Ruby Proc Documentation](https://docs.ruby-lang.org/en/4.0/Proc.html) — Official Ruby 4.0 reference for Proc

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*