---
source_course: "ruby"
source_lesson: "ruby-keyword-arguments"
---

# Keyword Arguments

## Introduction
Keyword arguments make method signatures self-documenting and eliminate the ambiguity of positional parameters. Modern Ruby (3.0+) enforces a strict separation between positional and keyword arguments, and Ruby 4.0 brings additional changes to splat behavior. This lesson covers keyword argument basics, double-splat capture, argument forwarding, and the Ruby 4.0 `*nil` change.

## Key Concepts
- **Keyword Argument**: A method parameter specified by name at the call site (`method(key: value)`), making calls self-documenting.
- **Default Value**: Keyword arguments can have defaults (`timeout: 30`), making them optional.
- **Double Splat (`**`)**: Captures any extra keyword arguments into a hash, similar to `*` for positional args.
- **Argument Forwarding (`...`)**: Ruby 3+ syntax that forwards all arguments (positional, keyword, and block) to another method.
- **Anonymous Block Forwarding (`&`)**: Ruby 3.1+ syntax for forwarding a block without naming it.

## Real World Context
Consider an HTTP client method: `get(url, timeout: 30, headers: {}, retries: 3)`. Keyword arguments make every call site readable — you always know what `30` means because it is labeled `timeout:`. The double splat is useful for options-hash patterns: a `configure(**options)` method that accepts arbitrary settings. Argument forwarding with `...` is essential for wrapper methods, middleware, and decorators that pass calls through transparently.

## Deep Dive
Modern Ruby enforces strict separation between positional and keyword arguments.

### Basic Keyword Arguments

Keyword arguments can have default values, making them optional at the call site:

```ruby
def call_api(url:, method: :get, timeout: 30)
  puts "#{method.upcase} #{url} (timeout: #{timeout})"
end

call_api(url: "/users")                # Uses defaults
call_api(url: "/users", method: :post)  # Override default
```

The `url:` parameter has no default, so it is required. `method:` and `timeout:` have defaults and are optional. This makes the method signature self-documenting.

### Capturing Extra Keywords

Use `**` (double splat) to capture additional keyword arguments into a hash:

```ruby
def request(url:, **options)
  puts "URL: #{url}"
  puts "Options: #{options}"
end

request(url: "/api", timeout: 30, retries: 3)
# URL: /api
# Options: {:timeout=>30, :retries=>3}
```

The `**options` captures all keyword arguments that don't match a named parameter. This is the modern replacement for the old `options = {}` hash parameter pattern.

### Forwarding Arguments

Ruby 3+ provides `...` for complete argument forwarding, passing positional args, keyword args, and the block all at once:

```ruby
def wrapper(...)
  puts "Before"
  inner_method(...)
  puts "After"
end

# Also forward blocks with anonymous &
def with_logging(&)
  puts "Start"
  yield
  puts "End"
end
```

The `...` syntax is ideal for decorator and middleware patterns where you want to pass everything through without enumerating each parameter. The anonymous `&` (Ruby 3.1+) forwards a block without assigning it a name.

### Ruby 4.0: No More *nil to Array Conversion

In Ruby 4.0, ``*nil` evaluates to an empty splat (no arguments), which has been consistent Ruby behavior, which is worth knowing when working with optional splat arguments:

```ruby
# Ruby 3.x: *nil => [] (called nil.to_a)
# Ruby 4.0: *nil => nil (no conversion)

# Be explicit when needed:
array = value.nil? ? [] : [*value]
```

If your code relied on `*nil` producing an empty array, you need to add explicit nil handling in Ruby 4.0.

## Common Pitfalls
1. **Passing a hash as the last positional argument** — In Ruby 3.0+, a hash literal as the last argument is no longer auto-converted to keyword arguments. You must use the explicit `**hash` double-splat if you want to pass a hash as keywords.
2. **Forgetting that `...` forwards everything** — The `...` forwarding syntax passes positional args, keyword args, *and* the block. If you only want to forward some arguments, use explicit parameters instead.

## Best Practices
1. **Use keyword arguments for any method with more than two parameters** — Positional arguments are fine for `min(a, b)`, but `create_user(name, email, role, active)` is much clearer as `create_user(name:, email:, role:, active:)`.
2. **Prefer `...` forwarding in wrapper methods** — When writing decorators, middleware, or logging wrappers, use `...` to forward arguments transparently. This avoids breaking when the wrapped method's signature changes.

## Summary
- Keyword arguments make method calls self-documenting with named parameters and optional defaults.
- Double splat (`**`) captures extra keywords; `...` forwards all arguments transparently.
- Ruby 4.0 changes `*nil` behavior — explicit nil handling is now required where `*nil` previously produced `[]`.

## Code Examples

**Keyword arguments with defaults and double splat for extra options**

```ruby
def call_api(url:, method: :get, timeout: 30, **headers)
  puts "#{method.upcase} #{url}"
  puts "Timeout: #{timeout}"
  puts "Headers: #{headers}" unless headers.empty?
end

call_api(url: "/users", method: :post, content_type: "application/json")
# POST /users
# Timeout: 30
# Headers: {:content_type=>"application/json"}
```


## Resources

- [Methods Syntax](https://docs.ruby-lang.org/en/4.0/syntax/methods_rdoc.html) — Official Ruby documentation covering method definitions and keyword arguments

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*