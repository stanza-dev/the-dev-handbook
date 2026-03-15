---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-testing-concurrency"
---

# Testing Concurrent Code

## Introduction
Concurrent code is notoriously difficult to test. Race conditions appear intermittently, timing-dependent bugs pass on fast machines and fail on slow CI servers, and non-deterministic scheduling makes reproduction nearly impossible. This lesson covers strategies for writing reliable, deterministic tests for concurrent Ruby code.

## Key Concepts
- **Non-determinism**: The property of concurrent code where the execution order varies between runs, making bugs appear and disappear unpredictably.
- **Deterministic testing**: Techniques that remove scheduling randomness so tests produce the same result every time.
- **Synchronization point**: A place in code where threads or fibers coordinate, allowing tests to control execution order.
- **Stress testing**: Running concurrent code under heavy load many times to surface rare race conditions.

## Real World Context
A CI pipeline that passes 99% of the time but fails randomly on a concurrent test is worse than no test at all — it trains the team to ignore failures. Writing deterministic concurrent tests eliminates flaky builds and gives you genuine confidence that your thread-safe code actually works.

## Deep Dive

### The Fundamental Challenge

Consider a simple thread-safe counter. A naive test looks correct but is unreliable:

```ruby
# Flaky test — timing-dependent
def test_counter_is_thread_safe
  counter = ThreadSafeCounter.new
  threads = 10.times.map do
    Thread.new { 1000.times { counter.increment } }
  end
  threads.each(&:join)
  assert_equal 10_000, counter.value  # Sometimes passes, sometimes fails!
end
```

This test might pass because the GVL serializes access on your fast development machine, but fail on CI where thread scheduling differs. The test does not actually verify thread safety — it verifies that the particular scheduling happened to be safe.

### Strategy 1: Use Queue for Synchronization

Replace timing assumptions with explicit synchronization using `Thread::Queue`:

```ruby
def test_producer_consumer
  queue = Thread::Queue.new
  results = []
  ready = Thread::Queue.new

  consumer = Thread.new do
    ready << :go  # Signal that consumer is running
    while (item = queue.pop) != :done
      results << item * 2
    end
  end

  ready.pop  # Wait until consumer is ready
  queue << 1
  queue << 2
  queue << 3
  queue << :done

  consumer.join
  assert_equal [2, 4, 6], results
end
```

The `ready` queue eliminates the race between starting the consumer and pushing items. Every run produces the same result because the test controls the order of operations.

### Strategy 2: Extract and Test the Logic Separately

Separate the concurrent orchestration from the business logic. Test the logic synchronously, then test the orchestration with minimal concurrent code:

```ruby
# Pure logic — easy to test
class OrderProcessor
  def process(order)
    validate(order)
    calculate_total(order)
    apply_discount(order)
  end
end

# Concurrent wrapper — thin layer
class ConcurrentOrderProcessor
  def initialize(pool_size:)
    @queue = Thread::Queue.new
    @processor = OrderProcessor.new
    @workers = pool_size.times.map { spawn_worker }
  end

  private

  def spawn_worker
    Thread.new do
      while (order = @queue.pop) != :stop
        @processor.process(order)
      end
    end
  end
end
```

Now `OrderProcessor` is tested with fast, deterministic unit tests. The concurrent wrapper only needs a small integration test.

### Strategy 3: Stress Testing for Race Conditions

When you need to verify thread safety, run many iterations to increase the probability of surfacing races:

```ruby
def test_thread_safety_under_stress
  100.times do |trial|
    counter = ThreadSafeCounter.new
    threads = 20.times.map do
      Thread.new { 500.times { counter.increment } }
    end
    threads.each(&:join)
    assert_equal 10_000, counter.value,
      "Race condition detected on trial #{trial}"
  end
end
```

Running the test 100 times with 20 threads each dramatically increases the chance of catching a race. If this test passes consistently, you have strong evidence (though not proof) of thread safety.

### Strategy 4: Timeout Guards

Concurrent tests can hang forever if a deadlock occurs. Always add a timeout:

```ruby
def test_no_deadlock
  result = nil
  thread = Thread.new { result = potentially_deadlocking_operation }

  # Fail fast instead of hanging CI forever
  completed = thread.join(5)  # 5-second timeout
  assert completed, "Operation deadlocked — thread did not finish in 5 seconds"
  assert_equal expected_value, result
end
```

The `join(timeout)` variant returns `nil` if the thread does not finish within the specified seconds, letting the test fail with a clear message instead of hanging.

## Common Pitfalls
1. **Using `sleep` for synchronization in tests** — `sleep(0.1)` is a guess, not a guarantee. Use Queue, Mutex, or ConditionVariable to synchronize threads deterministically.
2. **Testing thread safety with a single thread** — A test that runs concurrent code on one thread proves nothing about thread safety. Always use multiple threads in your test.
3. **Ignoring flaky test failures** — If a concurrent test fails even once, there is a real bug. Investigate immediately rather than re-running and hoping it passes.

## Best Practices
1. **Keep concurrent tests isolated** — Each test should create its own threads, queues, and shared state. Never share concurrent resources between test cases.
2. **Add timeouts to every concurrent test** — A 5-10 second timeout catches deadlocks and prevents CI from hanging indefinitely.
3. **Run stress tests in CI** — Configure your CI pipeline to run thread-safety stress tests with a higher iteration count than local development.

## Summary
- Concurrent tests are unreliable when they depend on scheduling order or timing.
- Use `Thread::Queue` as a synchronization primitive to make tests deterministic.
- Separate business logic from concurrent orchestration to maximize testable surface area.
- Stress tests with many iterations and threads increase confidence in thread safety.
- Always add timeouts to prevent deadlocked tests from hanging CI.

## Code Examples

**A fully deterministic concurrent test — Queue-based synchronization guarantees the same result on every run, with a timeout guard to catch deadlocks**

```ruby
# Deterministic concurrent test using Queue for synchronization
def test_worker_processes_all_items
  input_queue = Thread::Queue.new
  output_queue = Thread::Queue.new

  worker = Thread.new do
    while (item = input_queue.pop) != :done
      output_queue << item.upcase
    end
    output_queue << :finished
  end

  # Feed items in a controlled order
  input_queue << "alpha"
  input_queue << "beta"
  input_queue << "gamma"
  input_queue << :done

  # Collect results deterministically
  results = []
  while (result = output_queue.pop) != :finished
    results << result
  end

  worker.join(5)  # Timeout guard
  assert_equal ["ALPHA", "BETA", "GAMMA"], results
end
```


## Resources

- [Thread::Queue Documentation](https://docs.ruby-lang.org/en/4.0/Thread/Queue.html) — Official reference for Ruby's thread-safe Queue class used for inter-thread communication and test synchronization

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*