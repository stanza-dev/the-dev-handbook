---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-ractor-sharing"
---

# Data Isolation

Ractors cannot share mutable objects. Data must be:
1. **Shareable** (immutable/frozen)
2. **Copied** (deep copy for mutable objects)
3. **Moved** (original becomes inaccessible)

## Shareable Objects

```ruby
# Automatically shareable:
# - Integers, Symbols, true, false, nil
# - Frozen strings
# - Ractor objects themselves
# - Ractor::Port objects

r = Ractor.new do
  Ractor.receive
end

r.send(42)              # OK - Integer is shareable
r.send(:symbol)         # OK - Symbol is shareable
r.send("str".freeze)    # OK - Frozen string is shareable
```

## Copying Data

```ruby
# Mutable objects are deep-copied
data = { name: "Alice" }

r = Ractor.new do
  received = Ractor.receive
  received[:name] = "Bob"  # Modifying the COPY
  received
end

r.send(data)
r.value  # => { name: "Bob" }
data     # => { name: "Alice" } (original unchanged)
```

## Moving Data

```ruby
large_array = (1..1_000_000).to_a

r = Ractor.new do
  data = Ractor.receive
  data.sum
end

# Move instead of copy (more efficient)
r.send(large_array, move: true)

large_array  # Ractor::MovedError! Can't access anymore
```

## Make Objects Shareable

```ruby
# Explicitly make shareable (deeply freezes)
config = Ractor.make_shareable({ api_key: "xyz" })
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*