---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-debugging-tools"
---

# Building Your Own Tools

Combining metaprogramming techniques to create debugging utilities.

## Method Call Logger

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

## Object Change Tracker

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

## Time Travel Debugger (Concept)

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

## Performance Warning

TracePoint significantly impacts performance. Use only in development/debugging!

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*