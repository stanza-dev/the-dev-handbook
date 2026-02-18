---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-tracepoint"
---

# TracePoint

TracePoint lets you hook into Ruby VM events like method calls, line execution, and class definitions.

## Basic Usage

```ruby
trace = TracePoint.new(:call) do |tp|
  puts "Called: #{tp.defined_class}##{tp.method_id}"
end

trace.enable
"hello".upcase  # "Called: String#upcase"
trace.disable
```

## Event Types

```ruby
:line       # Line of code about to execute
:call       # Ruby method call
:return     # Ruby method return
:c_call     # C method call
:c_return   # C method return
:raise      # Exception raised
:b_call     # Block entry
:b_return   # Block exit
:class      # Class/module definition
:end        # Class/module definition end
```

## TracePoint Data

```ruby
TracePoint.new(:call, :return) do |tp|
  puts tp.event          # :call or :return
  puts tp.lineno         # Source line number
  puts tp.path           # Source file
  puts tp.method_id      # Method name
  puts tp.defined_class  # Class where method is defined
  puts tp.self           # Current self
  puts tp.binding        # Current binding
  puts tp.return_value   # Only for :return events
end
```

## Scoped Tracing

```ruby
# Trace only specific code
result = TracePoint.trace(:call) do |tp|
  puts tp.method_id
end

# Or with a block
TracePoint.trace(:call) { |tp| puts tp.method_id }.enable do
  # Only traces within this block
  some_method
end
```

## Resources

- [TracePoint Class](https://docs.ruby-lang.org/en/4.0/TracePoint.html) — TracePoint documentation

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*