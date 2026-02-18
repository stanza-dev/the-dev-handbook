---
source_course: "ruby"
source_lesson: "ruby-exception-hierarchy"
---

# Ruby's Exception Tree

Understanding the exception hierarchy is crucial for proper error handling:

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

## Never Rescue Exception

Rescuing `Exception` traps system signals like `SystemExit` and `Interrupt`:

```ruby
# BAD - catches Ctrl+C, kill signals, etc.
begin
  do_something
rescue Exception => e
  # This catches EVERYTHING
end

# GOOD - only catches StandardError and subclasses
begin
  do_something
rescue => e  # Defaults to StandardError
  puts e.message
end
```

## Specific Rescues

```ruby
begin
  risky_operation
rescue ArgumentError => e
  puts "Bad argument: #{e.message}"
rescue IOError => e
  puts "I/O problem: #{e.message}"
rescue => e
  puts "Other error: #{e.message}"
end
```

## Resources

- [Exception Class](https://docs.ruby-lang.org/en/4.0/Exception.html) — Ruby Exception hierarchy

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*