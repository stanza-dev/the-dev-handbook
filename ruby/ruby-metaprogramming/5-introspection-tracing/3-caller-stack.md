---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-caller-stack"
---

# Stack Traces & Caller

## Introduction

Ruby gives you direct access to the call stack through `Kernel#caller` and `Kernel#caller_locations`. These methods let you inspect which methods called the current one, where they are defined, and on which line. Understanding the call stack is critical for building loggers, deprecation warnings, and debugging utilities.

## Key Concepts

- **`caller`**: Returns the call stack as an array of formatted strings like `"file.rb:10:in 'method_name'"`
- **`caller_locations`**: Returns the call stack as an array of `Thread::Backtrace::Location` objects with structured access to path, line number, and label
- **Stack depth parameters**: Both methods accept `(start, length)` arguments to limit how much of the stack they return

## Real World Context

When you mark a method as deprecated, you want the warning message to include the location of the code that called the deprecated method, not the location of the deprecation warning itself. `caller_locations(1, 1).first` gives you exactly that: the caller's file and line number, which you can include in the warning so developers know where to update their code.

## Deep Dive

### Kernel#caller

The `caller` method returns an array of strings representing the call stack. Each string includes the file, line number, and method name:

```ruby
def deep_method
  caller  # Returns array of strings
end

def middle_method
  deep_method
end

def outer_method
  middle_method
end

outer_method
# => ["example.rb:6:in `middle_method'",
#     "example.rb:10:in `outer_method'",
#     "example.rb:13:in `<main>'"]
```

The array is ordered from the most recent caller to the outermost frame. The current method itself is not included.

### caller_locations (More Structured)

`caller_locations` returns `Thread::Backtrace::Location` objects, which provide structured access instead of string parsing. This example shows the available attributes:

```ruby
def show_stack
  caller_locations.each do |loc|
    puts "#{loc.path}:#{loc.lineno} in #{loc.label}"
  end
end

# Returns Thread::Backtrace::Location objects
loc = caller_locations.first
loc.path      # File path
loc.lineno    # Line number
loc.label     # Method name
loc.base_label  # Method name without decoration
```

Always prefer `caller_locations` over `caller` when you need to extract individual fields. Parsing the string format of `caller` is fragile and error-prone.

### Limiting Stack Depth

Both methods accept start and length arguments to control output. This is important for performance since capturing the full stack is expensive:

```ruby
# Only get first 5 frames
caller(0, 5)
caller_locations(0, 5)

# Skip first 2 frames
caller(2)
caller_locations(2)
```

Passing `(1, 1)` is the most common pattern: it skips the current method (frame 0) and returns only the immediate caller.

### Practical Uses

Here are two common applications of call stack inspection. First, adding caller context to log messages:

```ruby
# Logging with context
def log(message)
  loc = caller_locations(1, 1).first
  puts "[#{loc.path}:#{loc.lineno}] #{message}"
end

# Finding who called a method
def deprecated_method
  warn "Called from: #{caller_locations(1, 1).first}"
end
```

Both patterns use `caller_locations(1, 1)` to efficiently grab just the immediate caller without capturing the entire stack.

## Common Pitfalls

1. **Capturing the full stack in hot paths** — `caller` and `caller_locations` without arguments capture the entire stack, which is expensive. Always pass `(start, length)` in performance-sensitive code.
2. **Parsing `caller` strings with regex** — The string format can vary across Ruby versions and platforms. Use `caller_locations` instead for structured access.

## Best Practices

1. **Use `caller_locations` over `caller`** — Structured `Location` objects are safer and more readable than string parsing. They also have better performance characteristics.
2. **Always limit stack depth with `(start, length)`** — Pass `caller_locations(1, 1)` when you only need the immediate caller, or `(0, 10)` when you need a short backtrace.

## Summary

- `caller` returns the call stack as formatted strings; `caller_locations` returns structured `Location` objects
- Both accept `(start, length)` arguments to limit depth for better performance
- Common use cases include deprecation warnings, contextual logging, and custom error reporting

## Code Examples

**A deprecation warning helper that includes the caller's file and line number**

```ruby
module Deprecation
  def self.warn(message)
    loc = caller_locations(1, 1).first
    Kernel.warn "[DEPRECATED] #{message} (called from #{loc.path}:#{loc.lineno})"
  end
end

def old_method
  Deprecation.warn("old_method is deprecated, use new_method instead")
end

old_method
# [DEPRECATED] old_method is deprecated, use new_method instead (called from example.rb:9)
```


## Resources

- [Kernel#caller](https://docs.ruby-lang.org/en/4.0/Kernel.html#method-i-caller) — Official documentation for Kernel#caller and caller_locations with stack trace inspection

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*