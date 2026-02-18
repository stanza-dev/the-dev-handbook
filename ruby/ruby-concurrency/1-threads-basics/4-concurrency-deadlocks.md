---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-deadlocks"
---

# Deadlock Conditions

Deadlock occurs when threads wait for each other forever:

```ruby
mutex_a = Mutex.new
mutex_b = Mutex.new

# Thread 1
Thread.new do
  mutex_a.synchronize do
    sleep 0.1
    mutex_b.synchronize { puts "Thread 1" }  # Waiting for B
  end
end

# Thread 2
Thread.new do
  mutex_b.synchronize do
    sleep 0.1
    mutex_a.synchronize { puts "Thread 2" }  # Waiting for A
  end
end

# DEADLOCK! Both threads wait forever
```

## Prevention Strategies

### 1. Lock Ordering

```ruby
def transfer(from, to, amount)
  # Always lock in consistent order
  first, second = [from, to].sort_by(&:object_id)

  first.lock.synchronize do
    second.lock.synchronize do
      from.withdraw(amount)
      to.deposit(amount)
    end
  end
end
```

### 2. Try Lock with Timeout

```ruby
if mutex.try_lock
  begin
    # Critical section
  ensure
    mutex.unlock
  end
else
  # Couldn't acquire lock, try later
end
```

### 3. Avoid Nested Locks

```ruby
# Better: use one lock for related resources
class Account
  TRANSFER_LOCK = Mutex.new

  def self.transfer(from, to, amount)
    TRANSFER_LOCK.synchronize do
      from.withdraw(amount)
      to.deposit(amount)
    end
  end
end
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*