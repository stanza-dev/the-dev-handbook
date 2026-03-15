---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-binding-objects"
---

# The Binding Object

## Introduction

A `Binding` object captures the entire execution context at a point in your program — local variables, `self`, and the current block. This lesson explains how bindings work, how they power template engines like ERB, and why they must be handled with care.

## Key Concepts

- **Binding**: An object that encapsulates the execution context (local variables, `self`, block) at the point where `binding` was called.
- **eval with binding**: Evaluates a string or expression in the context captured by a `Binding`, giving access to that scope's variables.
- **TOPLEVEL_BINDING**: A constant that holds the binding of Ruby's top-level scope (`main`).

## Real World Context

ERB templates work by evaluating `<%= ... %>` expressions inside a binding you provide. When Rails renders a view, it passes the controller's binding so that instance variables like `@users` are accessible in the template. Understanding bindings explains how template engines bridge the gap between your code and your markup.

## Deep Dive

A `Binding` object captures the execution context at a specific point in your program:

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

Even though `get_binding` has returned, the `Binding` object `b` still has access to `local_var` and `param`. The binding keeps those variables alive.

You can also create variables through a binding:

```ruby
b = binding
b.local_variable_set(:x, 100)
b.eval("x * 2")  # => 200
```

Calling `local_variable_set` injects a variable into the binding's scope, and subsequent `eval` calls can see it.

### Ruby 4.0 Changes to Binding

In Ruby 4.0, `Binding#local_variables` no longer includes numbered block parameters (`_1`, `_2`, ...) or the `it` keyword. These are now accessed through new dedicated methods:

```ruby
# Using numbered parameters (_1)
[1, 2, 3].each do
  b = binding
  b.local_variables              # => [] (implicit params excluded)
  b.implicit_parameters          # => [:_1]
  b.implicit_parameter_get(:_1)  # => current element
  b.implicit_parameter_defined?(:_1)  # => true
end

# Using the it keyword (Ruby 3.4+)
[1, 2, 3].each do
  b = binding
  b.implicit_parameters          # => [:it]
  b.implicit_parameter_get(:it)  # => current element
end
```

Note that `_1` and `it` are mutually exclusive — a block uses one or the other, never both.

This change makes `local_variables` return only explicitly declared variables, improving clarity when inspecting bindings.

ERB is the most common real-world use of bindings. The template evaluates expressions in the context of the binding you provide:

```ruby
require 'erb'

name = "Ruby"
template = ERB.new("Hello, <%= name %>!")
template.result(binding)  # => "Hello, Ruby!"
```

The `binding` call captures the current scope (including the `name` variable), and ERB evaluates `name` inside that context.

Ruby also provides access to the top-level binding:

```ruby
TOPLEVEL_BINDING.eval("self")  # => main
```

This can be useful for evaluating code in a clean top-level scope.

Bindings expose internal state, so you must never evaluate untrusted input:

```ruby
# DANGEROUS!
user_input = "system('rm -rf /')"  # Malicious
binding.eval(user_input)  # Would execute!
```

Any string passed to `eval` runs as Ruby code with full access to the binding's context, making it a serious security risk if the input is not trusted.

## Common Pitfalls

1. **Evaluating user-supplied strings** — Never pass untrusted input to `binding.eval`. It executes arbitrary Ruby code with full access to local variables and system calls.
2. **Assuming bindings are lightweight** — Each `Binding` keeps its entire scope alive, preventing garbage collection of captured variables. Creating many bindings in a loop can cause memory bloat.

## Best Practices

1. **Prefer block-based eval over string eval** — Use `instance_eval { }` with a block instead of `eval("string")` whenever possible. Blocks are safer, faster, and get syntax highlighting.
2. **Limit binding exposure** — When passing a binding to a template engine, consider using a dedicated object with only the variables the template needs, rather than exposing your entire scope.

## Summary

- A `Binding` captures the full execution context (variables, `self`, block) at a specific point.
- ERB and similar template engines use bindings to evaluate expressions in your code's context.
- Never evaluate untrusted strings through a binding — it grants full code execution.

## Code Examples

**Using a binding to inject variables into an ERB template**

```ruby
require 'erb'

def render(template_str, vars = {})
  b = binding
  vars.each { |k, v| b.local_variable_set(k, v) }
  ERB.new(template_str).result(b)
end

render("Hello, <%= name %>!", name: "World")
# => "Hello, World!"
```


## Resources

- [Binding Class](https://docs.ruby-lang.org/en/4.0/Binding.html) — Official Ruby documentation for the Binding class, covering eval, local_variable_get, and local_variable_set.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*