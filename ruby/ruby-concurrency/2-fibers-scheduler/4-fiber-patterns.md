---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-fiber-patterns"
---

# Common Fiber Patterns

## 1. Coroutine Communication

```ruby
def producer
  Fiber.new do
    items = ["a", "b", "c"]
    items.each { |item| Fiber.yield item }
    nil  # Signal completion
  end
end

def consumer(producer_fiber)
  while (item = producer_fiber.resume)
    puts "Consuming: #{item}"
  end
end

consumer(producer)
```

## 2. Pipeline

```ruby
def stage1(input)
  Fiber.new do
    input.each { |x| Fiber.yield x * 2 }
  end
end

def stage2(fiber)
  Fiber.new do
    while (value = fiber.resume)
      Fiber.yield value + 1
    end
  end
end

pipeline = stage2(stage1([1, 2, 3]))
while (result = pipeline.resume)
  puts result  # 3, 5, 7
end
```

## 3. State Machine

```ruby
def traffic_light
  Fiber.new do
    loop do
      Fiber.yield :green
      Fiber.yield :yellow
      Fiber.yield :red
    end
  end
end

light = traffic_light
6.times { puts light.resume }  # green, yellow, red, green, yellow, red
```

## 4. Timeout with Fiber

```ruby
def with_timeout(seconds)
  fiber = Fiber.current
  timer = Thread.new do
    sleep seconds
    fiber.raise(Timeout::Error) if fiber.alive?
  end

  begin
    yield
  ensure
    timer.kill
  end
end
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*