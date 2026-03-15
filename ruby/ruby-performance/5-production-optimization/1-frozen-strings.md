---
source_course: "ruby-performance"
source_lesson: "ruby-performance-frozen-strings"
---

# Frozen String Literals

## Introduction
Every time Ruby encounters a string literal like `"hello"`, it allocates a brand-new String object on the heap. In a hot loop or a frequently-called method, this means thousands of identical, throwaway allocations that pressure the garbage collector. The `# frozen_string_literal: true` magic comment tells Ruby to freeze every string literal in the file, eliminating duplicate allocations entirely.

## Key Concepts
- **Frozen String**: A string that cannot be mutated. Calling `<<`, `gsub!`, or any bang method on it raises `FrozenError`.
- **Magic Comment**: The `# frozen_string_literal: true` pragma at the top of a file. It must appear on the first line (or after a shebang/encoding comment) to take effect.
- **String Deduplication**: Ruby can reuse a single object for identical frozen string literals instead of allocating new copies each time.
- **`String#-@` (unary minus)**: The `-"hello"` syntax returns a frozen, deduplicated copy of the string from Ruby's internal fstring table.

## Real World Context
Rails applications routinely process thousands of requests per second. Each request may evaluate the same string literal hundreds of times across controllers, serializers, and views. Without frozen strings, a single endpoint can allocate tens of thousands of short-lived String objects per request, increasing minor GC pauses and memory usage. Enabling frozen string literals across your codebase is one of the simplest and most impactful performance wins in production Ruby.

## Deep Dive

### The Allocation Problem

Consider a method called on every request:

```ruby
def content_type
  "application/json"  # New String allocated every call
end
```

Each invocation creates a fresh String object. Over 10,000 requests, that is 10,000 identical objects the GC must track and eventually sweep.

With the magic comment, every string literal in the file is automatically frozen:

```ruby
# frozen_string_literal: true

def content_type
  "application/json"  # Frozen and deduplicated — same object every time
end
```

Now Ruby returns the same frozen String object on every call. Zero additional allocations.

### Deduplication with `String#-@`

If you cannot add the magic comment to an entire file, you can freeze and deduplicate individual strings with the unary minus operator:

```ruby
key = -"cache_key"  # Returns a frozen, deduplicated string
# Equivalent to: "cache_key".freeze (but also deduplicates)
```

The difference between `.freeze` and `-@` is subtle but important. `.freeze` freezes the receiver in place, while `-@` returns a frozen copy from Ruby's internal fstring (frozen string) table, ensuring only one copy exists in memory across the entire process.

### The `-W:performance` Warning Flag

Ruby provides a warning flag that identifies string literals that would benefit from freezing:

```bash
ruby -W:performance my_app.rb
```

This flag emits warnings when Ruby detects string literals that are allocated repeatedly without freezing. It is invaluable for auditing a codebase before adopting frozen string literals.

### Ruby 4.0: Lock-Free Frozen String Table

In Ruby 4.0, the internal frozen string table has been redesigned to be lock-free. In earlier Rubies, deduplicating frozen strings required acquiring a global lock, which became a bottleneck in multi-Ractor programs. The lock-free implementation means `String#-@` and frozen literal deduplication scale linearly across Ractors without contention.

## Common Pitfalls
1. **Mutating a frozen string** — Adding `# frozen_string_literal: true` to a file that calls `<<` or `gsub!` on string literals will raise `FrozenError` at runtime. Audit your code for in-place mutations before enabling the pragma. Use `.dup` when you need a mutable copy: `+"hello"` or `"hello".dup`.
2. **Assuming `freeze` deduplicates** — Calling `.freeze` on a string freezes it but does not guarantee deduplication. Use `-@` (unary minus) or the magic comment for true deduplication from the fstring table.
3. **Placing the magic comment in the wrong position** — The comment must be on the very first line of the file (or immediately after `#!` or `# encoding:` lines). Placing it after a `require` statement means it is ignored silently.

## Best Practices
1. **Enable globally with Rubocop** — Use `Style/FrozenStringLiteralComment: EnforcedStyle: always` in your `.rubocop.yml` to ensure every file has the magic comment. This catches omissions during code review.
2. **Use unary plus for mutable strings** — When you need a mutable string in a frozen-literal file, use `+"template"` to get an unfrozen duplicate. This is more idiomatic than `.dup` and clearly signals intent.
3. **Run `-W:performance` in CI** — Add the performance warning flag to your test suite to catch string allocation regressions before they reach production.

## Summary
- `# frozen_string_literal: true` freezes all string literals in a file, eliminating redundant allocations.
- `String#-@` (unary minus) freezes and deduplicates a string via Ruby's fstring table.
- Ruby 4.0 makes the frozen string table lock-free, removing contention in multi-Ractor programs.
- Use `-W:performance` to audit your codebase for unfrozen string allocations.
- Always audit for in-place string mutations before enabling the pragma.

## Code Examples

**Enabling frozen string literals file-wide and selectively creating mutable copies with unary plus (+) or deduplicating with unary minus (-)**

```ruby
# frozen_string_literal: true

# All string literals in this file are frozen and deduplicated
def headers
  {
    "Content-Type"  => "application/json",   # Frozen — same object every call
    "X-Request-Id"  => SecureRandom.uuid      # Method call, not a literal — still mutable
  }
end

# When you need a mutable copy, use unary plus
mutable = +"hello"
mutable << " world"  # Works fine — unary plus returns unfrozen dup

# Unary minus deduplicates without the magic comment
key = -"session_token"  # Frozen + deduplicated from fstring table
```


## Resources

- [Frozen String Literal Pragma](https://docs.ruby-lang.org/en/4.0/syntax/comments_rdoc.html) — Ruby documentation on magic comments including frozen_string_literal

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*