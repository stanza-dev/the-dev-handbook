---
source_course: "ruby"
source_lesson: "ruby-exception-hierarchy"
---

# The Exception Hierarchy

## Introduction
Ruby organizes all errors into a class hierarchy rooted at `Exception`. Understanding this tree is the first step to writing robust error handling, because rescuing the wrong class can make your program impossible to interrupt or debug.

## Key Concepts
- **Exception**: The root class of all Ruby errors. You should almost never rescue this directly.
- **StandardError**: The default class rescued by bare `rescue`. Your custom errors should inherit from this.
- **RuntimeError**: The default error raised by `raise` with just a string message.

## Real World Context
Imagine you deploy a long-running background worker. If you rescue `Exception` instead of `StandardError`, pressing Ctrl+C or sending a SIGTERM to gracefully shut down the process will be swallowed by your rescue block. Your process becomes unkillable without `kill -9`, and deployments grind to a halt.

## Deep Dive

Ruby's exception hierarchy is a class tree. Every rescuable error descends from `Exception`, but the ones you typically care about live under `StandardError`.

Here is the top-level structure of the hierarchy:

```
Exception
├── NoMemoryError
├── ScriptError
├── SignalException
├── SystemExit
└── StandardError     <-- rescue this!
    ├── ArgumentError
    ├── IOError
    ├── NameError
    ├── RuntimeError   <-- default raise type
    ├── TypeError
    └── (your custom errors)
```

The key insight is that `StandardError` and its children represent errors your application can reasonably recover from, while everything else represents system-level signals that should propagate.

### Never Rescue Exception

Rescuing `Exception` traps system signals like `SystemExit` and `Interrupt`. The following example shows the dangerous pattern:

```ruby
# BAD - catches Ctrl+C, kill signals, etc.
begin
  do_something
rescue Exception => e
  # This catches EVERYTHING
end
```

This is dangerous because your code will silently swallow signals the operating system uses to manage your process. Instead, use a bare `rescue` which defaults to `StandardError`:

```ruby
# GOOD - only catches StandardError and subclasses
begin
  do_something
rescue => e  # Defaults to StandardError
  puts e.message
end
```

Notice that the bare `rescue => e` is equivalent to `rescue StandardError => e`. This is idiomatic Ruby.

### Specific Rescues

You can rescue multiple exception types in order from most specific to least specific:

```ruby
begin
  risky_operation
rescue ArgumentError => e
  puts "Bad argument: \#{e.message}"
rescue IOError => e
  puts "I/O problem: \#{e.message}"
rescue => e
  puts "Other error: \#{e.message}"
end
```

Ruby checks rescue clauses top-to-bottom and runs the first one that matches. Always put more specific exceptions first, otherwise they will be shadowed by a broader rescue above them.

## Common Pitfalls
1. **Rescuing Exception instead of StandardError** — This catches `Interrupt`, `SystemExit`, and `NoMemoryError`, making your process nearly impossible to stop gracefully. Always use bare `rescue` or explicitly rescue `StandardError`.
2. **Ordering rescue clauses from broad to narrow** — If you put `rescue StandardError` before `rescue ArgumentError`, the specific clause will never execute because `ArgumentError` is a subclass of `StandardError`.

## Best Practices
1. **Default to bare rescue** — A bare `rescue => e` catches `StandardError` and its subclasses, which is almost always what you want.
2. **Rescue the narrowest class possible** — Catching only the errors you expect makes debugging easier and prevents masking unexpected bugs.

## Summary
- Ruby's exception hierarchy has `Exception` at the root, but you should rescue `StandardError` (or its subclasses) in application code.
- Never rescue `Exception` directly — it traps system signals and makes your process hard to manage.
- Order rescue clauses from most specific to least specific so that each handler gets a chance to match.

## Code Examples

**Shows how to inspect the ancestor chain of exception classes and demonstrates a safe, specific rescue pattern.**

```ruby
# Demonstrating the exception hierarchy
puts ArgumentError.ancestors
# => [ArgumentError, StandardError, Exception, Object, Kernel, BasicObject]

puts SignalException.ancestors
# => [SignalException, Exception, Object, Kernel, BasicObject]

# Safe rescue pattern
begin
  Integer("not_a_number")
rescue ArgumentError => e
  puts "Caught: #{e.message}"
  # => Caught: invalid value for Integer(): "not_a_number"
end
```


## Resources

- [Exception Class](https://docs.ruby-lang.org/en/4.0/Exception.html) — Ruby Exception hierarchy

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*