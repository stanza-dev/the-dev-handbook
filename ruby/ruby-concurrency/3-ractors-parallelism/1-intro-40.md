---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-ractors-intro-40"
---

# Ractors in Ruby 4.0

## Introduction
Ractors bring true parallelism to Ruby by giving each Ractor its own Global VM Lock. Unlike threads, which share a single GVL and cannot execute Ruby code simultaneously, Ractors run on separate CPU cores with fully isolated interpreter state. Ruby 4.0 redesigned the Ractor API, removing `Ractor.yield` and `Ractor#take` in favor of Ports, `join`, and `value`.

## Key Concepts
- **Ractor**: An actor-like concurrency primitive with its own GVL, enabling true parallel execution of Ruby code.
- **Isolation**: Each Ractor has its own heap segment; mutable objects cannot be shared across Ractor boundaries.
- **`join`**: Blocks the caller until the Ractor terminates, similar to `Thread#join`.
- **`value`**: Returns the Ractor block's return value after it terminates.
- **Default Port**: Every Ractor has a built-in `Ractor::Port` for receiving messages via `Ractor.receive`.

## Real World Context
CPU-bound workloads like image processing, data transformation pipelines, and cryptographic operations cannot benefit from Ruby threads due to the GVL. Ractors solve this by running truly parallel Ruby code. Production systems use Ractors to saturate all available CPU cores for batch jobs, background computation, and request-level parallelism in custom servers.

## Deep Dive

Creating a Ractor is similar to creating a thread. You pass a block that runs in an isolated context:

```ruby
# Create a Ractor that performs a heavy computation
worker = Ractor.new do
  (1..1_000_000).reduce(:+)
end

# Wait for the Ractor to finish and retrieve its result
worker.join
result = worker.value  # => 500000500000
```

The block runs on a separate OS thread with its own GVL, so it does not block other Ractors or the main Ractor. `join` blocks until termination, and `value` returns whatever the block returned.

You can also send messages to a Ractor's default port using `send` and receive them inside with `Ractor.receive`:

```ruby
encoder = Ractor.new do
  message = Ractor.receive
  message.upcase
end

encoder.send("hello")
encoder.join
encoder.value  # => "HELLO"
```

The `send` method delivers the message to the Ractor's default port. Inside the Ractor, `Ractor.receive` pulls the next message from that port. Since `"hello"` is a mutable String, Ruby deep-copies it across the boundary automatically.

### Key Class Methods

`Ractor.current` returns the Ractor object for the currently executing Ractor. `Ractor.main` returns the main Ractor (the one running your script). `Ractor.count` returns how many Ractors are alive:

```ruby
puts Ractor.main == Ractor.current  # => true (in main script)

workers = 4.times.map { Ractor.new { sleep 1 } }
puts Ractor.count  # => 5 (main + 4 workers)
workers.each(&:join)
```

### Removed API

The following methods from Ruby 3.x no longer exist in Ruby 4.0:
- `Ractor.yield` — replaced by sending to a `Ractor::Port`
- `Ractor#take` — replaced by `Ractor#join` + `Ractor#value`
- `Ractor#close_incoming` / `Ractor#close_outgoing` — replaced by `Ractor::Port#close`

## Common Pitfalls
1. **Calling `value` before `join`** — `value` implicitly joins, but calling `join` first makes intent clear and lets you handle termination errors separately from value retrieval.
2. **Assuming shared memory** — Unlike threads, Ractors cannot access variables from the enclosing scope. Any data the Ractor needs must be passed as arguments to `Ractor.new` or sent via a port.

## Best Practices
1. **Pass initialization data as arguments** — Use `Ractor.new(arg1, arg2) { |a1, a2| ... }` to pass data at creation time rather than sending it afterward, which reduces coordination complexity.
2. **Name your Ractors** — Use the `name:` keyword for debugging: `Ractor.new(name: "image-processor") { ... }`.
3. **Prefer `value` for simple computations** — When a Ractor just computes and returns a result, `Ractor.new { compute }.value` is the cleanest pattern.

## Summary
- Ractors provide true parallelism by giving each one its own GVL.
- `Ractor.new(*args) { block }` creates a new Ractor; pass data as block arguments.
- `join` waits for termination; `value` retrieves the return value.
- `Ractor.current`, `Ractor.main`, and `Ractor.count` inspect running Ractors.
- The old `yield`/`take`/`close_incoming`/`close_outgoing` API is removed in Ruby 4.0.

## Code Examples

**Splitting a CPU-bound prime-finding task across four Ractors — each processes its own range in true parallel**

```ruby
# Parallel prime checking across 4 Ractors
ranges = [(2..2500), (2501..5000), (5001..7500), (7501..10000)]

workers = ranges.map do |range|
  Ractor.new(range, name: "primes-#{range}") do |r|
    r.select { |n| (2..Math.sqrt(n)).none? { |d| n % d == 0 } }
  end
end

primes = workers.flat_map { |w| w.join; w.value }
puts "Found #{primes.size} primes up to 10000"  # => Found 1229 primes
```


## Resources

- [Ractor Class (Ruby 4.0)](https://docs.ruby-lang.org/en/4.0/Ractor.html) — Official Ruby 4.0 Ractor API reference covering creation, messaging, and lifecycle

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*