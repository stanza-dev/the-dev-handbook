---
source_course: "ruby"
source_lesson: "ruby-symbols-strings"
---

# Symbols: Immutable Identifiers

Symbols are immutable identifiers, often used as hash keys and method names. Unlike strings, the same symbol literal always refers to the same object in memory.

```ruby
:hello.object_id == :hello.object_id  # => true
"hello".object_id == "hello".object_id # => false (usually)
```

## When to Use Symbols

**Use symbols for:**
- Hash keys (especially in options)
- Method names and references
- Enum-like values
- Anything that represents a "name" rather than "text"

```ruby
# Hash keys
user = { name: "Alice", age: 30 }
user[:name] # => "Alice"

# Method references
method_name = :upcase
"hello".send(method_name) # => "HELLO"

# Enum-like values
status = :pending
```

## Converting Between Symbols and Strings

```ruby
:hello.to_s    # => "hello"
"hello".to_sym # => :hello
```

## Memory Efficiency

Symbols are stored in a global symbol table. Creating many unique symbols can lead to memory bloat since they're never garbage collected (though Ruby 2.2+ did improve dynamic symbol GC).

## Resources

- [Symbol Class](https://docs.ruby-lang.org/en/4.0/Symbol.html) — Official Symbol documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*