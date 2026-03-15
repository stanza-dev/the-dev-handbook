---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-debugging-tools"
---

# Building Debugging Tools

## Introduction

This lesson brings together the metaprogramming techniques from this course to build practical debugging utilities. By combining TracePoint, method introspection, and dynamic method definition, you can create powerful tools tailored to your application. These tools are for development and testing only — never deploy them to production.

## Key Concepts

- **Method call logger**: A tool that wraps methods to print entry/exit information including arguments and return values
- **Change tracker**: An observer that detects when object state changes between method calls
- **Time travel debugger**: A concept for recording execution state at every line, allowing backward inspection

## Real World Context

When debugging a complex data transformation pipeline, adding `puts` statements to every method is tedious and easy to forget. A method call logger that you can attach to an entire class with one line gives you full visibility into the call sequence, arguments, and return values without modifying any source code. This is the same approach that tools like `ruby-spy` and `binding.pry`'s step debugger use internally.

## Deep Dive

### Method Call Logger

The following tool wraps any method to log its arguments and return value. It uses `instance_method` to capture the original, then `define_method` to replace it:

```ruby
module MethodLogger
  def self.watch(klass, method_name)
    original = klass.instance_method(method_name)

    klass.define_method(method_name) do |*args, &block|
      puts ">> #{klass}##{method_name}(#{args.inspect})"
      result = original.bind(self).call(*args, &block)
      puts "<< #{result.inspect}"
      result
    end
  end
end

MethodLogger.watch(String, :upcase)
"hello".upcase
# >> String#upcase([])
# << "HELLO"
```

The key technique is `original.bind(self).call(...)`, which calls the original unbound method in the context of the current object. This avoids `alias_method` and keeps the class clean.

### Object Change Tracker

This tool takes a snapshot of an object's instance variables and can report what changed. It demonstrates singleton class manipulation:

```ruby
module ChangeTracker
  def self.track(obj)
    obj.instance_variables.each do |ivar|
      original_value = obj.instance_variable_get(ivar)

      obj.singleton_class.define_method(:changes) do
        @_changes ||= {}
      end

      # Track on next assignment...
    end
  end
end
```

The tracker defines a `changes` method on the object's singleton class, so only the tracked object gets this method. Other instances of the same class are unaffected.

### Time Travel Debugger (Concept)

By combining TracePoint with binding capture, you can record the state of local variables at every line of execution. This is a conceptual implementation:

```ruby
class TimeMachine
  def initialize
    @snapshots = []
    @trace = TracePoint.new(:line) do |tp|
      @snapshots << {
        line: tp.lineno,
        file: tp.path,
        binding: tp.binding,
        locals: tp.binding.local_variables.map { |v|
          [v, tp.binding.local_variable_get(v)]
        }.to_h
      }
    end
  end

  def record(&block)
    @trace.enable(&block)
    @snapshots
  end
end
```

Each snapshot captures the file, line number, and all local variables at that point. After recording, you can inspect `@snapshots` to see exactly how state evolved. Note that this is extremely expensive and should only be used on small code blocks.

### Performance Warning

TracePoint significantly impacts performance. Use these tools only in development and debugging environments, never in production.

## Common Pitfalls

1. **Accidentally deploying debugging tools to production** — Wrapping methods with loggers or enabling TracePoint in production causes severe performance degradation. Always guard these tools behind environment checks.
2. **Capturing bindings in long-running processes** — Stored bindings hold references to all local variables, preventing garbage collection. Clear snapshots after inspection to avoid memory leaks.

## Best Practices

1. **Use `UnboundMethod#bind` instead of `alias_method`** — As shown in the MethodLogger, capturing the original method as an `UnboundMethod` and rebinding it is cleaner and does not pollute the class namespace.
2. **Scope TracePoint to the smallest possible block** — Use `trace.enable { ... }` to record only the code you care about. This minimizes both overhead and noise in the output.

## Summary

- Method call loggers use `instance_method` + `define_method` to non-invasively wrap any method
- Object change trackers leverage singleton classes to add monitoring to individual objects
- TracePoint-based recording captures line-by-line execution state but must be scoped carefully for performance

## Code Examples

**An extended MethodLogger that can instrument all instance methods of a class at once**

```ruby
module MethodLogger
  def self.watch_all(klass)
    klass.instance_methods(false).each do |method_name|
      watch(klass, method_name)
    end
  end

  def self.watch(klass, method_name)
    original = klass.instance_method(method_name)
    klass.define_method(method_name) do |*args, &block|
      puts ">> #{klass}##{method_name}(#{args.map(&:inspect).join(', ')})"
      result = original.bind(self).call(*args, &block)
      puts "<< #{result.inspect}"
      result
    end
  end
end

# Usage: MethodLogger.watch_all(MyService)
```

**A time travel debugger that records local variable state at each line of execution**

```ruby
class TimeMachine
  def initialize
    @snapshots = []
  end

  def record(&block)
    trace = TracePoint.new(:line) do |tp|
      @snapshots << {
        line: tp.lineno,
        locals: tp.binding.local_variables.map { |v|
          [v, tp.binding.local_variable_get(v)]
        }.to_h
      }
    end
    trace.enable(&block)
    @snapshots
  end
end

# snapshots = TimeMachine.new.record { your_code_here }
```


## Resources

- [Kernel#pp](https://docs.ruby-lang.org/en/4.0/Kernel.html#method-i-pp) — Official documentation for Kernel#pp and related debugging output methods

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*