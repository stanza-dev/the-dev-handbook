---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-ractor-sharing"
---

# Sharing Data Between Ractors

## Introduction
Ractors enforce memory isolation: each Ractor has its own heap, and mutable objects cannot cross boundaries. This isolation is what makes true parallelism safe — no data races, no need for locks. But it also means you must understand how data moves between Ractors: copying, moving, or sharing frozen objects.

## Key Concepts
- **Shareable Object**: An immutable (deeply frozen) object that multiple Ractors can reference simultaneously. Integers, Symbols, `true`, `false`, `nil`, frozen Strings, Ractor objects, and Ractor::Port objects are all shareable.
- **Copying**: When you send a mutable object, Ruby deep-copies it so each Ractor has its own independent copy.
- **Moving (`move: true`)**: Transfers ownership — the sender loses access and the receiver gets the original object. More efficient than copying for large data.
- **`Ractor.make_shareable(obj)`**: Deeply freezes an object graph, making it shareable across all Ractors.
- **`Ractor.shareable?(obj)`**: Returns `true` if the object can be shared without copying.

## Real World Context
In a parallel web scraper, each Ractor processes different URLs. Configuration data (API keys, rate limits) must be shared across all Ractors. You freeze the config once with `Ractor.make_shareable` and pass it to every worker. Large response bodies, on the other hand, are moved rather than copied to avoid doubling memory usage.

## Deep Dive

### Automatically Shareable Types

Certain types are always shareable without any action:

```ruby
Ractor.shareable?(42)            # => true (Integer)
Ractor.shareable?(:status)       # => true (Symbol)
Ractor.shareable?(true)          # => true (Boolean)
Ractor.shareable?(nil)           # => true (NilClass)
Ractor.shareable?("hello".freeze)  # => true (frozen String)
```

These types are immutable by nature (or frozen explicitly), so multiple Ractors can read them concurrently without risk.

### Deep Copying

Mutable objects are deep-copied when sent to a Ractor. The sender retains its original:

```ruby
original = { users: ["Alice", "Bob"], count: 2 }

worker = Ractor.new do
  data = Ractor.receive
  data[:users] << "Charlie"  # Modifies the COPY only
  data
end

worker.send(original)
modified = worker.value  # => { users: ["Alice", "Bob", "Charlie"], count: 2 }
original[:users]          # => ["Alice", "Bob"] — unchanged
```

Ruby walks the entire object graph and duplicates every reachable mutable object. This is safe but can be expensive for large data structures.

### Moving Data

For large objects, moving avoids the copy overhead by transferring ownership:

```ruby
large_dataset = (1..1_000_000).to_a

worker = Ractor.new do
  data = Ractor.receive
  data.sum
end

worker.send(large_dataset, move: true)
# large_dataset is now inaccessible!

begin
  large_dataset.size
rescue Ractor::MovedError => e
  puts e.message  # => can not send any methods to a moved object
end

worker.value  # => 500000500000
```

After a move, any access to the original raises `Ractor::MovedError`. This guarantees no two Ractors ever hold the same mutable reference.

### Making Objects Shareable

`Ractor.make_shareable` deeply freezes an object and all its nested references:

```ruby
config = {
  database: { host: "db.example.com", port: 5432 },
  api_keys: ["key_abc", "key_def"]
}

Ractor.make_shareable(config)
config.frozen?                    # => true
config[:database].frozen?         # => true
config[:api_keys][0].frozen?      # => true
Ractor.shareable?(config)         # => true

# Now safe to pass to any number of Ractors
5.times { Ractor.new(config) { |c| puts c[:database][:host] } }
```

Pass `copy: true` to freeze a copy instead of the original: `Ractor.make_shareable(obj, copy: true)`.

### Shareable Procs with `Ractor.shareable_lambda`

Ruby 4.0 introduces `Ractor.shareable_lambda` and `Ractor.shareable_proc` for creating Proc objects that can be shared across Ractor boundaries:

```ruby
# Create a shareable lambda
transform = Ractor.shareable_lambda { |x| x * 2 + 1 }
Ractor.shareable?(transform)  # => true

# Pass the lambda to multiple Ractors
results = 4.times.map do |i|
  Ractor.new(i, transform) do |num, fn|
    fn.call(num)
  end
end

results.map(&:value)  # => [1, 3, 5, 7]
```

Normal Procs and lambdas capture their enclosing scope and cannot be shared. `Ractor.shareable_lambda` creates a lambda that is isolated from its lexical environment — it can only reference shareable values.

## Common Pitfalls
1. **Capturing outer variables in Ractor blocks** — The `Ractor.new { }` block cannot close over local variables from the outer scope. Pass them as arguments: `Ractor.new(data) { |d| ... }`. Accessing outer locals raises `Ractor::IsolationError`.
2. **Forgetting that `make_shareable` mutates the original** — It freezes the object in place. If you still need a mutable copy, use `Ractor.make_shareable(obj, copy: true)`.

## Best Practices
1. **Freeze configuration at boot** — Call `Ractor.make_shareable` on config hashes, lookup tables, and constant data once during application startup so all Ractors can share them without copying.
2. **Move large temporary data** — When a Ractor produces a large intermediate result that will only be consumed by one downstream Ractor, move it instead of copying to halve memory usage.
3. **Use `shareable_lambda` for reusable transformations** — When multiple Ractors need the same pure function, wrap it in `Ractor.shareable_lambda` instead of duplicating the logic.

## Summary
- Ractors enforce memory isolation: no shared mutable state.
- Mutable objects are deep-copied by default; use `move: true` to transfer ownership.
- `Ractor.make_shareable(obj)` deeply freezes an object for cross-Ractor sharing.
- `Ractor.shareable?(obj)` checks if an object can be shared.
- `Ractor.shareable_lambda { }` creates Proc objects safe to share between Ractors.
- Accessing a moved object raises `Ractor::MovedError`.

## Code Examples

**Combining make_shareable for frozen data with shareable_lambda for reusable logic — both passed safely to parallel Ractors**

```ruby
# Sharing a frozen lookup table across multiple Ractors
currency_rates = Ractor.make_shareable({
  "USD" => 1.0, "EUR" => 0.92, "GBP" => 0.79, "JPY" => 149.5
})

converter = Ractor.shareable_lambda { |amount, from, to|
  amount * currency_rates[to] / currency_rates[from]
}

orders = [
  { amount: 100, from: "USD", to: "EUR" },
  { amount: 250, from: "GBP", to: "JPY" }
]

results = orders.map do |order|
  Ractor.new(order, converter) do |o, fn|
    fn.call(o[:amount], o[:from], o[:to])
  end
end

results.each { |r| puts r.value.round(2) }  # => 92.0, 47312.5 (approx)
```


## Resources

- [Ractor Shareable Objects](https://docs.ruby-lang.org/en/4.0/Ractor.html#class-Ractor-label-Shareable+objects) — Official docs on shareable objects, make_shareable, and isolation rules

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*