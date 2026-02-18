---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-ractors-intro-40"
---

# Ractors: True Parallelism

Ractors allow parallel execution by isolating interpreter state. Each Ractor has its own GVL!

## Ruby 4.0 API Changes

**Removed methods:**
- `Ractor.yield` - Use Ports instead
- `Ractor#take` - Use Ports instead
- `Ractor#close_incoming` / `close_outgoing`

**New methods:**
- `Ractor#join` - Wait for termination
- `Ractor#value` - Get return value
- `Ractor::Port` - New communication primitive

## Basic Ractor (Ruby 4.0)

```ruby
# Create a Ractor that computes something
r = Ractor.new do
  # This runs in parallel!
  heavy_computation
end

# Wait for completion and get result
r.join    # Blocks until Ractor terminates
r.value   # Returns the Ractor's return value

# Or combine:
result = Ractor.new { compute }.value
```

## Sending Messages (Ruby 4.0)

```ruby
# Each Ractor has a default port
r = Ractor.new do
  # Receive from default port
  msg = Ractor.receive
  msg.upcase
end

# Send to Ractor's default port
r.send("hello")
r.value  # => "HELLO"
```

## Why Ractors?

- **True parallelism**: No GVL sharing between Ractors
- **Memory safety**: No shared mutable state
- **Scales with cores**: Each Ractor can run on different CPU

## Resources

- [Ractor Class](https://docs.ruby-lang.org/en/4.0/Ractor.html) — Ractor documentation

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*