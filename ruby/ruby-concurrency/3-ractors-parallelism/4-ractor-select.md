---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-ractor-select"
---

# Ractor.select (Ruby 4.0)

Wait for any of multiple Ractors/Ports to be ready.

## Updated Select API

```ruby
# Ruby 4.0: select only accepts Ractors and Ports
ractors = 3.times.map do |i|
  Ractor.new(i) { |n| sleep(rand); n * 2 }
end

# Wait for first to complete
ready_ractor, result = Ractor.select(*ractors)
puts "Ractor #{ready_ractor} returned #{result}"
```

## Selecting with Ports

```ruby
port1 = Ractor::Port.new
port2 = Ractor::Port.new

# Start producers
Ractor.new(port1) { |p| sleep 0.1; p.send("from 1") }
Ractor.new(port2) { |p| sleep 0.2; p.send("from 2") }

# Wait for first message
ready_port, message = Ractor.select(port1, port2)
puts message  # "from 1" (arrives first)
```

## Worker Pool Pattern

```ruby
def parallel_map(items, workers: 4)
  result_port = Ractor::Port.new

  items.each_slice((items.size / workers.to_f).ceil).map do |chunk|
    Ractor.new(chunk, result_port) do |data, port|
      results = data.map { |item| yield(item) }
      port.send(results)
    end
  end

  results = []
  workers.times do
    _, chunk_results = Ractor.select(result_port)
    results.concat(chunk_results)
  end
  results
end

parallel_map([1, 2, 3, 4, 5, 6, 7, 8]) { |n| n * 2 }
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*