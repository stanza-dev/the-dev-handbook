---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-ractor-ports"
---

# Ractor::Port (Ruby 4.0)

Ports are the new primary way to communicate between Ractors.

## Creating and Using Ports

```ruby
# Create a custom port
port = Ractor::Port.new

# Start a Ractor that receives from port
worker = Ractor.new(port) do |p|
  loop do
    msg = p.receive
    break if msg == :stop
    puts "Got: #{msg}"
  end
end

# Send messages through port
port.send("hello")
port.send("world")
port.send(:stop)

worker.join
```

## Default Port

Each Ractor has a default port:

```ruby
r = Ractor.new do
  # Get default port
  Ractor.current.default_port

  # Receive from default port
  Ractor.receive  # Same as Ractor.current.default_port.receive
end

# Send to Ractor's default port
r.send("message")  # Same as r.default_port.send("message")
```

## Port Methods

```ruby
port = Ractor::Port.new

port.send(obj)      # Send an object
port.receive        # Blocking receive
port.close          # Close the port
```

## Multiple Ports Pattern

```ruby
input_port = Ractor::Port.new
output_port = Ractor::Port.new

worker = Ractor.new(input_port, output_port) do |inp, out|
  while (data = inp.receive)
    result = process(data)
    out.send(result)
  end
end

input_port.send(job)
result = output_port.receive
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*