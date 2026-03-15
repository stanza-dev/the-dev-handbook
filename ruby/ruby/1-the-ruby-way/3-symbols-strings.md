---
source_course: "ruby"
source_lesson: "ruby-symbols-strings"
---

# Symbols vs Strings

## Introduction

Ruby has two text-like types: symbols and strings. They look similar but serve very different purposes. Symbols are immutable identifiers optimized for comparison, while strings are mutable sequences of characters meant for text manipulation. Choosing the right one impacts both code clarity and performance.

## Key Concepts

- **Symbol**: An immutable, interned identifier (e.g., `:name`). The same symbol literal always refers to the same object in memory.
- **String**: A mutable sequence of characters (e.g., `\"hello\"`). Each string literal creates a new object by default.
- **Symbol table**: A global registry where Ruby stores all symbols for fast lookup and identity comparison.

## Real World Context

In a Rails application, you use symbols constantly: as hash keys in params (`params[:user]`), as method names for dynamic dispatch (`send(:validate)`), and as configuration identifiers (`status: :active`). Using strings where symbols belong wastes memory and slows comparisons, while using symbols where strings belong prevents garbage collection.

## Deep Dive

Symbols are immutable identifiers, often used as hash keys and method names. Unlike strings, the same symbol literal always refers to the same object in memory.

You can verify this identity behavior with `object_id`:

```ruby
:hello.object_id == :hello.object_id  # => true
\"hello\".object_id == \"hello\".object_id # => false (usually)
```

The symbol `:hello` always has the same `object_id`, while each `\"hello\"` string is a distinct object.

### When to Use Symbols

Use symbols for hash keys (especially in options), method names and references, enum-like values, and anything that represents a \"name\" rather than \"text\":

```ruby
# Hash keys
user = { name: \"Alice\", age: 30 }
user[:name] # => \"Alice\"

# Method references
method_name = :upcase
\"hello\".send(method_name) # => \"HELLO\"

# Enum-like values
status = :pending
```

In each case, the symbol acts as a label or name rather than textual content that you would manipulate.

### Converting Between Symbols and Strings

Ruby makes it easy to convert between the two types:

```ruby
:hello.to_s    # => \"hello\"
\"hello\".to_sym # => :hello
```

These conversions are useful when interfacing with APIs that expect one type but you have the other.

### Memory Efficiency

Symbols are stored in a global symbol table. Creating many unique symbols can lead to memory bloat since they are never garbage collected (though Ruby 2.2+ did improve dynamic symbol GC for symbols created from strings at runtime).

## Common Pitfalls

1. **Creating symbols from user input** — Converting untrusted user input to symbols with `to_sym` can lead to a symbol table denial-of-service attack, since static symbols (written literally in code) are never garbage collected, while dynamic symbols created at runtime can be GC'd since Ruby 2.2. Always validate or limit the set of allowed symbols.
2. **Mixing symbol and string keys in hashes** — `{ name: \"Alice\" }` uses the symbol key `:name`, but `{ \"name\" => \"Alice\" }` uses a string key. Accessing with the wrong type returns nil. Use `Hash#transform_keys` or Rails' `HashWithIndifferentAccess` to normalize.

## Best Practices

1. **Use symbols for identifiers, strings for data** — If the value represents a name, key, or label, use a symbol. If it represents user-facing text or data that might be manipulated, use a string.
2. **Prefer the modern hash syntax for symbol keys** — Write `{ name: \"Alice\" }` instead of `{ :name => \"Alice\" }`. The colon syntax is cleaner and signals that the key is a symbol.

## Summary

- Symbols are immutable identifiers that share a single object in memory; strings are mutable and create new objects.
- Use symbols for hash keys, method names, and enum-like values; use strings for text data.
- Be cautious about creating symbols from untrusted input due to the non-garbage-collected symbol table.

## Code Examples

**Shows the difference between symbols and strings in identity, hash keys, and conversion**

```ruby
# Symbols are immutable and share identity
:hello.object_id == :hello.object_id  # => true
"hello".object_id == "hello".object_id # => false

# Symbol keys in hashes (modern syntax)
user = { name: "Alice", age: 30 }
user[:name]    # => "Alice"

# Converting between types
:hello.to_s    # => "hello"
"hello".to_sym # => :hello
```


## Resources

- [Symbol Class](https://docs.ruby-lang.org/en/4.0/Symbol.html) — Official Symbol documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*