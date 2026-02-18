---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-binding-objects"
---

# Binding

A `Binding` object encapsulates the execution context (variables, methods, self, block) at a specific point.

```ruby
def get_binding(param)
  local_var = "local"
  binding
end

b = get_binding("argument")

b.local_variable_get(:local_var)  # => "local"
b.local_variable_get(:param)      # => "argument"
b.eval("local_var + ' + ' + param")  # => "local + argument"
```

## Creating Variables via Binding

```ruby
b = binding
b.local_variable_set(:x, 100)
b.eval("x * 2")  # => 200
```

## ERB Uses Bindings

```ruby
require 'erb'

name = "Ruby"
template = ERB.new("Hello, <%= name %>!")
template.result(binding)  # => "Hello, Ruby!"
```

## TOPLEVEL_BINDING

Ruby provides access to the top-level binding:

```ruby
TOPLEVEL_BINDING.eval("self")  # => main
```

## Security Warning

Bindings expose internal state. Never eval untrusted input:

```ruby
# DANGEROUS!
user_input = "system('rm -rf /')"  # Malicious
binding.eval(user_input)  # Would execute!
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*