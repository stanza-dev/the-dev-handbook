---
source_course: "ruby"
source_lesson: "ruby-procs-vs-lambdas"
---

# Two Types of Closures

Both Procs and Lambdas are instances of `Proc`, but they differ in two key ways:

## 1. Argument Handling (Arity)

**Lambdas** are strict about arguments:

```ruby
lam = ->(a, b) { a + b }
lam.call(1)         # ArgumentError!
lam.call(1, 2)      # => 3
lam.call(1, 2, 3)   # ArgumentError!
```

**Procs** are lenient:

```ruby
proc = Proc.new { |a, b| [a, b] }
proc.call(1)        # => [1, nil]
proc.call(1, 2)     # => [1, 2]
proc.call(1, 2, 3)  # => [1, 2] (extra ignored)
```

## 2. Return Behavior

**Lambdas**: `return` exits the lambda:

```ruby
def test_lambda
  lam = -> { return 10 }
  lam.call
  puts "After lambda"  # This runs!
end
test_lambda # prints "After lambda"
```

**Procs**: `return` exits the *enclosing method*:

```ruby
def test_proc
  pr = Proc.new { return 10 }
  pr.call
  puts "After proc"  # Never reached!
end
test_proc  # Returns 10, no output
```

## Creating Lambdas

```ruby
# Arrow syntax (preferred)
lam = ->(x) { x * 2 }

# Traditional syntax
lam = lambda { |x| x * 2 }
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*