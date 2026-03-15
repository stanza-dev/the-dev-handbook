---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-tracepoint"
---

# TracePoint

## Introduction

TracePoint lets you hook into Ruby VM events such as method calls, returns, line execution, and exception raises. It is the built-in instrumentation API that makes debuggers, profilers, and coverage tools possible. This lesson teaches you how to create, enable, and scope trace points for effective runtime introspection.

## Key Concepts

- **TracePoint**: An object that subscribes to VM events and executes a callback block when those events fire
- **Event types**: Symbols like `:call`, `:return`, `:line`, `:raise` that specify which VM events to observe
- **Scoped tracing**: Enabling a trace only within a specific block of code to minimize performance overhead

## Real World Context

When debugging a production issue where a method is being called from an unexpected location, you can temporarily enable a TracePoint on `:call` for that specific method to log every call site. Profiling tools like `ruby-prof` and coverage tools like `simplecov` rely on TracePoint (or its C-level equivalent) under the hood.

## Deep Dive

### Basic Usage

You create a TracePoint by specifying one or more event types and a callback block. The following example traces all Ruby method calls:

```ruby
trace = TracePoint.new(:call) do |tp|
  puts "Called: #{tp.defined_class}##{tp.method_id}"
end

trace.enable
"hello".upcase  # "Called: String#upcase"
trace.disable
```

The trace must be explicitly enabled and disabled. Between those calls, every matching event triggers the callback.

### Event Types

Ruby supports a comprehensive set of event types. Here is the full list:

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

Choose the narrowest event type for your use case. Tracing `:line` events generates enormous output and overhead, while `:call` is more targeted.

### TracePoint Data

Inside the callback block, the `tp` parameter provides rich context about the event. Here are the available attributes:

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

Note that `tp.return_value` is only available for `:return` and `:c_return` events. Accessing it during a `:call` event raises a `RuntimeError`.

### Scoped Tracing

To minimize performance impact, you can enable tracing for a specific block only. This example traces method calls within a limited scope:

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

Scoped tracing ensures that the callback is only active during the block execution, keeping overhead contained.

## Common Pitfalls

1. **Leaving traces enabled in production** — TracePoint adds significant overhead. Always disable traces after use and never ship enabled trace points to production.
2. **Triggering infinite recursion in callbacks** — If your callback calls a method that itself triggers the trace event, you get infinite recursion. Use a guard flag or disable the trace inside the callback.

## Best Practices

1. **Use scoped `enable` with a block** — This guarantees the trace is disabled when the block exits, even if an exception is raised. It is the TracePoint equivalent of `ensure`.
2. **Filter events by class or method in the callback** — Rather than tracing everything, check `tp.defined_class` or `tp.method_id` early in the callback and return unless it matches your target.

## Summary

- TracePoint hooks into Ruby VM events like `:call`, `:return`, `:raise`, and `:line`
- It provides rich context (file, line, class, method, binding) inside the callback block
- Always scope traces to minimize performance impact and disable them when done

## Code Examples

**A simple profiler that counts method calls during a block and reports the top 5 most-called methods**

```ruby
# Count method calls per class during a block
counts = Hash.new(0)
trace = TracePoint.new(:call) do |tp|
  counts["#{tp.defined_class}##{tp.method_id}"] += 1
end

trace.enable do
  [3, 1, 2].sort.map(&:to_s)
end

counts.sort_by { |_, v| -v }.first(5).each do |method, count|
  puts "#{method}: #{count}"
end
```


## Resources

- [TracePoint Class](https://docs.ruby-lang.org/en/4.0/TracePoint.html) — TracePoint documentation

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*