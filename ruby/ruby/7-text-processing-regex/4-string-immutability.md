---
source_course: "ruby"
source_lesson: "ruby-string-immutability"
---

# String Immutability & Performance

## Introduction
Strings are mutable by default in Ruby, which means every string literal creates a new object in memory. The frozen string literal pragma and explicit freezing let you opt into immutability for significant memory savings and safer code. Ruby 3.4 introduced \"chilled strings\" as the first step toward making frozen string literals the default in a future version. Understanding when and how to freeze strings is essential for writing performant Ruby.

## Key Concepts
- **Mutable string**: A string that can be modified in place with methods like `<<` and `upcase!`. This is Ruby's default behavior.
- **Frozen string**: A string marked as immutable. Any attempt to modify it raises a `FrozenError`.
- **Chilled string** (Ruby 3.4+): A string literal without the pragma that emits a deprecation warning when mutated, signaling a future shift toward frozen-by-default.
- **frozen_string_literal pragma**: A magic comment (`# frozen_string_literal: true`) that makes all string literals in a file frozen by default.
- **String deduplication**: Ruby's ability to reuse the same object for identical frozen strings, reducing memory usage.

## Real World Context
A Rails application handling thousands of requests per second creates millions of string objects for things like hash keys, SQL fragments, and HTTP headers. Most of these strings are identical across requests and never modified. Freezing them lets Ruby reuse a single object instead of allocating a new one each time, which can reduce memory usage by 10-20% in string-heavy applications.

## Deep Dive

### The Frozen String Pragma

Adding the magic comment at the top of a file makes all string literals in that file frozen:

```ruby
# frozen_string_literal: true

str = "hello"
str << " world"  # => FrozenError!

# To modify, create a new string:
str = str + " world"  # New object
str = "\#{str} world"  # Interpolation creates new object
```

When a frozen string is modified, Ruby raises `FrozenError` immediately, making it easy to find code that incorrectly mutates strings. The `+` operator and string interpolation create new string objects, so they work fine with frozen strings.

### Chilled Strings (Ruby 3.4+)

Starting in Ruby 3.4, string literals in files *without* the `frozen_string_literal` pragma are \"chilled\": they are technically mutable, but mutating them emits a deprecation warning. This is Ruby's migration path toward making frozen strings the default in a future version:

```ruby
# No pragma — Ruby 3.4+ chilled strings
str = "hello"
str << " world"  # Works, but with -W:deprecated prints:
# warning: literal string will be frozen in the future
```

Note: chilled string warnings only appear when deprecation warnings are enabled (via `-W:deprecated` flag or `Warning[:deprecated] = true`). They are not emitted by default. To silence or prevent them, either add `# frozen_string_literal: true` (and fix mutation sites) or use `String.new("hello")` for strings you intend to mutate. This gives you time to prepare your codebase before a future Ruby version makes frozen literals the default.

### Performance Benefits

Without freezing, each string literal creates a new object even when the content is identical. With freezing, Ruby can reuse the same object:

```ruby
# Without frozen: each creates a NEW string object
1000.times { "hello" }  # 1000 String objects

# With frozen: same object reused
# frozen_string_literal: true
1000.times { "hello" }  # 1 String object
```

This difference is dramatic in hot loops and frequently called methods. A method called 10,000 times per second that contains three string literals would allocate 30,000 objects per second without freezing, versus 3 objects total with freezing.

### When to Use Frozen Strings

Frozen strings are a net positive in most situations, but there are cases where mutability is needed:

- **Do use** for new projects and libraries
- **Do use** for performance-critical code
- **Don't use** if you need to mutate strings often (e.g., building strings with `<<` in a loop)

For string building, prefer an array with `join` or use `StringIO` instead of mutating a string with `<<`.

### String Deduplication

Ruby 2.5+ automatically deduplicates frozen strings at the VM level. Two frozen strings with the same content share the same object:

```ruby
a = "hello".freeze
b = "hello".freeze
a.object_id == b.object_id  # => true
```

This deduplication happens automatically and means that freezing strings not only prevents mutation but actively reduces the number of objects in memory.

### Explicit Freezing

You can freeze individual strings without enabling the pragma for the entire file:

```ruby
# Freeze at assignment
CONSTANT = "important".freeze

# Or use the unary minus operator
CONSTANT = -"important"  # Same as .freeze
```

The unary minus operator (`-"string"`) is a concise way to freeze a string and is commonly used for constants and hash keys. It also triggers deduplication, so `-"hello"` always returns the same object.

## Common Pitfalls
1. **Using << on frozen strings** — The shovel operator mutates in place and will raise `FrozenError` on frozen strings. Use `+` or interpolation to create new strings instead.
2. **Forgetting the pragma only affects literals** — The `frozen_string_literal: true` pragma freezes string *literals* in the file. Strings created dynamically (e.g., from `String.new` or external input) are still mutable.
3. **Ignoring chilled string warnings** — Ruby 3.4+ emits deprecation warnings when string literals are mutated without the pragma. Fix these warnings now to prepare for a future Ruby version where frozen literals become the default.

## Best Practices
1. **Enable frozen_string_literal: true in all new files** — This is the default for many Ruby style guides and catches accidental mutation early. Many linters can add this automatically.
2. **Use the unary minus operator for inline freezing** — `-"string"` is more concise than `"string".freeze` and triggers deduplication, making it ideal for constants and hash keys.
3. **Use `String.new` for intentionally mutable strings** — When you need to build a string with `<<`, use `String.new` to make the intent explicit and avoid chilled-string warnings.

## Summary
- Strings are mutable by default in Ruby, creating a new object for each literal. The `frozen_string_literal: true` pragma makes all literals in a file immutable.
- Ruby 3.4+ introduced \"chilled strings\" that warn on mutation, as a migration path toward frozen-by-default in a future version.
- Frozen strings enable deduplication, where identical strings share the same object in memory, significantly reducing allocation pressure.
- Use `+` or interpolation instead of `<<` when working with frozen strings, and prefer `-"string"` for inline freezing.

## Code Examples

**Shows the frozen string literal pragma in action, including the FrozenError, safe alternatives, and object deduplication with the unary minus operator.**

```ruby
# frozen_string_literal: true

# Demonstrating frozen string behavior
greeting = "hello"

# This raises FrozenError:
# greeting << " world"

# These create new strings (safe):
new_greeting = greeting + " world"
puts new_greeting  # => hello world

# Deduplication proof:
a = -"hello"
b = -"hello"
puts a.object_id == b.object_id  # => true
puts a.frozen?                    # => true
```

**Benchmarks the allocation difference between mutable and frozen string literals.**

```ruby
# Performance comparison: frozen vs mutable strings
require 'benchmark'

Benchmark.bm(15) do |x|
  x.report('mutable:')  { 1_000_000.times { "hello" } }
  x.report('frozen:')   { 1_000_000.times { -"hello" } }
end
# frozen version allocates far fewer objects
```


## Resources

- [String Class](https://docs.ruby-lang.org/en/4.0/String.html) — Official Ruby documentation for the String class, including freezing behavior and the frozen_string_literal pragma.

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*