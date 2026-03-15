---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-deadlocks"
---

# Avoiding Deadlocks

## Introduction
A deadlock occurs when two or more threads are permanently stuck, each waiting for a lock held by the other. Deadlocks are among the hardest concurrency bugs to diagnose because the program simply hangs — no exception, no crash, no log entry. This lesson covers the four conditions that cause deadlocks and three strategies to prevent them.

## Key Concepts
- **Deadlock**: A state where two or more threads are each waiting for a resource held by the other, resulting in permanent blocking.
- **Lock Ordering**: A strategy where all threads acquire multiple locks in the same predetermined order, preventing circular wait.
- **try_lock**: A non-blocking lock acquisition attempt that returns `false` immediately if the lock is unavailable.

## Real World Context
Bank transfers are the classic deadlock scenario: transferring from Account A to Account B locks A then B, while a simultaneous transfer from B to A locks B then A. Without consistent lock ordering, both transfers deadlock. This pattern appears in any system where multiple resources must be updated atomically.

## Deep Dive

### How Deadlocks Happen

The following code demonstrates a classic deadlock. Thread 1 locks `vault_lock` then tries to lock `ledger_lock`, while Thread 2 locks `ledger_lock` then tries to lock `vault_lock`. Both threads wait forever.

```ruby
vault_lock = Mutex.new
ledger_lock = Mutex.new

# Thread 1: locks vault, then wants ledger
Thread.new do
  vault_lock.synchronize do
    sleep 0.1  # Gives Thread 2 time to lock ledger
    ledger_lock.synchronize { puts "Thread 1: transferred" }
  end
end

# Thread 2: locks ledger, then wants vault
Thread.new do
  ledger_lock.synchronize do
    sleep 0.1  # Gives Thread 1 time to lock vault
    vault_lock.synchronize { puts "Thread 2: transferred" }
  end
end

# DEADLOCK — both threads block forever
```

Neither thread can proceed because each holds a lock the other needs.

### Strategy 1: Consistent Lock Ordering

If every thread acquires locks in the same order, circular wait is impossible. Sort locks by a deterministic key like `object_id`.

```ruby
def transfer(from_account, to_account, amount)
  # Always lock the account with the smaller object_id first
  first, second = [from_account, to_account].sort_by(&:object_id)

  first.lock.synchronize do
    second.lock.synchronize do
      from_account.withdraw(amount)
      to_account.deposit(amount)
    end
  end
end
```

Because both directions of transfer acquire locks in the same order, deadlock cannot occur.

### Strategy 2: try_lock with Retry

Instead of blocking forever, `try_lock` returns immediately. If the lock is unavailable, release all held locks and retry.

```ruby
def safe_transfer(from_account, to_account, amount)
  loop do
    if from_account.lock.try_lock
      begin
        if to_account.lock.try_lock
          begin
            from_account.withdraw(amount)
            to_account.deposit(amount)
            return true
          ensure
            to_account.lock.unlock
          end
        end
      ensure
        from_account.lock.unlock
      end
    end
    sleep(rand * 0.01)  # Back off before retrying
  end
end
```

The random backoff prevents livelock, where both threads repeatedly fail at the same time.

### Strategy 3: Single Lock for Related Resources

The simplest approach: use one lock for all related operations. This eliminates nested locking entirely.

```ruby
class Bank
  TRANSFER_LOCK = Mutex.new

  def self.transfer(from_account, to_account, amount)
    TRANSFER_LOCK.synchronize do
      from_account.withdraw(amount)
      to_account.deposit(amount)
    end
  end
end
```

This trades some parallelism (only one transfer at a time) for simplicity and correctness. For many applications, this is the right tradeoff.

## Common Pitfalls
1. **Acquiring locks in inconsistent order** — The most common deadlock cause. If you must hold multiple locks, always acquire them in a deterministic, globally consistent order.
2. **Holding locks during I/O** — Network calls or disk reads inside a `synchronize` block hold the lock for unpredictable durations. Fetch data first, then lock only for the update.

## Best Practices
1. **Minimize lock scope** — Hold locks for the shortest time possible. Compute values outside the lock, then acquire it only to update shared state.
2. **Prefer single-lock designs** — Unless profiling proves you need finer-grained locking, a single lock per resource group is simpler and deadlock-free.

## Summary
- Deadlocks happen when threads wait for locks held by each other in a circular chain.
- Consistent lock ordering (sorted by `object_id`) prevents circular wait.
- `try_lock` with backoff avoids indefinite blocking.
- A single coarse-grained lock is the simplest deadlock-free design.

## Code Examples

**Two accounts transferred safely using lock ordering by object_id — prevents deadlock regardless of transfer direction**

```ruby
# Deadlock-free transfers using consistent lock ordering
class Account
  attr_reader :lock, :balance, :name

  def initialize(name, balance)
    @name = name
    @balance = balance
    @lock = Mutex.new
  end

  def withdraw(amount) = @balance -= amount
  def deposit(amount) = @balance += amount
end

def transfer(from, to, amount)
  first, second = [from, to].sort_by(&:object_id)
  first.lock.synchronize do
    second.lock.synchronize do
      from.withdraw(amount)
      to.deposit(amount)
    end
  end
end

checking = Account.new("Checking", 1000)
savings  = Account.new("Savings", 500)
transfer(checking, savings, 200)
# checking.balance => 800, savings.balance => 700
```


## Resources

- [Mutex Class](https://docs.ruby-lang.org/en/4.0/Thread/Mutex.html) — Ruby Mutex reference covering synchronize, try_lock, and lock methods

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*