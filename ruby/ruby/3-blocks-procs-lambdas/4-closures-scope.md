---
source_course: "ruby"
source_lesson: "ruby-closures-scope"
---

# Closures and Variable Scope

## Introduction

Blocks, Procs, and lambdas in Ruby are closures, meaning they capture and retain access to variables from the scope where they were defined. This powerful capability enables patterns like counters, memoization, and encapsulated state. Understanding how closures interact with variable scope is essential for writing correct and predictable Ruby code.

## Key Concepts

- **Closure**: A function that captures variables from its surrounding scope and retains access to them even after the scope has exited.
- **Binding**: An object that encapsulates the execution context (local variables, self, and block) at a particular point in the code.
- **Shared state**: Multiple closures defined in the same scope share references to the same variables, enabling coordinated behavior.

## Real World Context

Closures are the mechanism behind memoization in Ruby (`@result ||= expensive_computation`), callback handlers that need access to their creation context, and configuration DSLs where a block captures variables from the calling scope. In Rails, closures power scope definitions (`scope :active, -> { where(active: true) }`) and background job callbacks that reference local variables from the enqueueing context.

## Deep Dive

Blocks, Procs, and Lambdas are closures. They capture variables from their surrounding scope and retain access to them:

```ruby
def counter
  count = 0
  -> { count += 1 }  # Lambda captures 'count'
end

c = counter
c.call  # => 1
c.call  # => 2
c.call  # => 3
```

Each call to `c` increments the same `count` variable, even though the `counter` method has already returned. The lambda holds a reference to the variable, not a copy of its value.

### Sharing State Between Closures

Multiple closures defined in the same scope share references to the same variables, enabling coordinated behavior:

```ruby
def create_counter
  count = 0
  {
    increment: -> { count += 1 },
    decrement: -> { count -= 1 },
    value: -> { count }
  }
end

counter = create_counter
counter[:increment].call  # => 1
counter[:increment].call  # => 2
counter[:decrement].call  # => 1
counter[:value].call      # => 1
```

All three lambdas share the same `count` variable. This is similar to encapsulation in object-oriented programming, but achieved through closures alone.

### The Binding Object

Every closure has a `binding` that captures its execution context, including local variables:

```ruby
def get_binding(value)
  binding
end

b = get_binding(42)
b.local_variable_get(:value)  # => 42
b.eval(\"value * 2\")           # => 84
```

The `binding` object lets you introspect and evaluate code in the captured context. This is how ERB templates access variables from their rendering context.

### Closures Keep References, Not Values

Closures hold references to captured variables, which means they see the current value at the time of execution:

```ruby
lambdas = []
3.times do |i|
  lambdas << -> { i }
end
lambdas.map(&:call)  # => [0, 1, 2] - each captured its own 'i'
```

In this case, each iteration of `times` creates a new scope with its own `i` variable, so each lambda captures a different variable. This is safe because block parameters create new bindings per iteration.

## Common Pitfalls

1. **Assuming closures capture values, not references** — A closure sees the current value of a variable at call time, not at creation time. If the variable changes between creation and invocation, the closure sees the new value.
2. **Memory leaks from long-lived closures** — Closures keep their entire binding alive, which can prevent garbage collection of large objects referenced in the enclosing scope. Be mindful of what variables are in scope when creating long-lived Procs or lambdas.

## Best Practices

1. **Keep closure scopes small** — Define closures in the smallest scope necessary so they do not inadvertently capture large objects or irrelevant variables.
2. **Use closures for encapsulation** — When you need private state without a full class, a method that returns closures sharing a local variable is a clean and lightweight alternative.

## Summary

- Closures (blocks, Procs, lambdas) capture references to variables from their surrounding scope.
- Multiple closures in the same scope share the same variable references, enabling coordinated state management.
- The `binding` object lets you introspect a closure's captured execution context.

## Code Examples

**Shows how closures capture variables from their enclosing scope and how multiple closures share state**

```ruby
# Closures capture surrounding variables
def counter
  count = 0
  -> { count += 1 }
end

c = counter
c.call  # => 1
c.call  # => 2
c.call  # => 3

# Multiple closures share the same variable
def create_counter
  count = 0
  {
    increment: -> { count += 1 },
    decrement: -> { count -= 1 },
    value:     -> { count }
  }
end
```


## Resources

- [Ruby Proc Documentation](https://docs.ruby-lang.org/en/4.0/Proc.html) — Official Ruby 4.0 reference for Proc

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*