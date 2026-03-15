---
source_course: "ruby-performance"
source_lesson: "ruby-performance-memoization"
---

# Memoization & Caching

## Introduction
Memoization stores the result of an expensive computation so it is only calculated once. In Ruby, the `||=` operator is the idiomatic way to memoize, but there are subtleties around `nil`/`false` values, thread safety, and memory growth that every production developer must understand.

## Key Concepts
- **Memoization**: Caching the return value of a method so subsequent calls return the cached result without re-executing the method body.
- **`||=` (or-equals)**: The standard Ruby memoization operator. `@value ||= compute` assigns `compute` only if `@value` is `nil` or `false`.
- **Lazy Initialization**: Deferring object creation until the object is first needed, reducing startup cost and memory usage.
- **LRU Cache**: A cache with a maximum size that evicts the Least Recently Used entry when full, preventing unbounded memory growth.

## Real World Context
In a Rails application, a method like `current_user` might hit the database on every call within a request. Memoizing it with `@current_user ||= User.find(session[:user_id])` turns N database queries into one. However, memoizing at the class or module level (rather than instance level) can cause memory leaks that persist across requests, silently growing until the process is killed by the OOM killer.

## Deep Dive

### Basic Memoization with `||=`

The simplest and most common pattern:

```ruby
class Report
  def total_revenue
    @total_revenue ||= orders.sum(&:amount)
  end
end
```

The first call computes the sum and stores it in `@total_revenue`. All subsequent calls return the cached value instantly.

### The `nil`/`false` Trap

`||=` does not work correctly when the memoized value can legitimately be `nil` or `false`:

```ruby
class Feature
  def enabled?
    # BUG: if the result is false, this re-computes every time
    @enabled ||= Settings.check(:feature_flag)
  end
end
```

Since `false || Settings.check(...)` re-evaluates the right side, the memoization breaks. Use `defined?` or a sentinel instead:

```ruby
class Feature
  def enabled?
    return @enabled if defined?(@enabled)
    @enabled = Settings.check(:feature_flag)
  end
end
```

Now `@enabled` is cached regardless of whether it is `true`, `false`, or `nil`.

### Hash-Based Computed Caches

For methods that accept arguments, use `Hash.new` with a block to build a self-populating cache:

```ruby
class TaxCalculator
  def initialize
    @cache = Hash.new { |hash, region| hash[region] = compute_tax_rate(region) }
  end

  def tax_rate(region)
    @cache[region]  # Computed on first access, cached thereafter
  end

  private

  def compute_tax_rate(region)
    # Expensive API call or database lookup
    TaxApi.rate_for(region)
  end
end
```

The block is called automatically for any key not yet in the hash, making the cache self-populating and clean.

### LRU Cache for Bounded Memory

Unbounded caches grow forever. In long-running processes (Sidekiq workers, Puma servers), this leads to memory bloat. An LRU cache evicts the oldest entries:

```ruby
class LruCache
  def initialize(max_size: 1000)
    @max_size = max_size
    @data = {}
  end

  def fetch(key)
    if @data.key?(key)
      # Move to end (most recently used) by re-inserting
      @data[key] = @data.delete(key)
    else
      @data[key] = yield
      @data.delete(@data.first[0]) if @data.size > @max_size
    end
    @data[key]
  end
end

cache = LruCache.new(max_size: 500)
result = cache.fetch("user:42") { User.find(42) }
```

Ruby's Hash preserves insertion order (since Ruby 1.9), so deleting and re-inserting moves a key to the end. When the cache exceeds `max_size`, the first (oldest) key is evicted.

### When Memoization Hurts

Memoization trades memory for speed. It hurts when:

- The cached value is large and the method is called with many distinct arguments (cache grows without bound).
- The cached value becomes stale (e.g., memoizing a database result across requests in a class variable).
- The method is only called once anyway, so the overhead of the instance variable check is pure waste.

Always ask: "How many distinct values will this cache hold, and for how long?"

## Common Pitfalls
1. **Memoizing `nil`/`false` with `||=`** — As shown above, `||=` re-evaluates when the cached value is falsy. Use `defined?` or a sentinel value to handle these cases.
2. **Class-level memoization leaks** — Storing memoized values in class variables (`@@cache`) or module-level instance variables means the cache persists across requests and grows without bound. Prefer instance-level memoization scoped to the request lifecycle.
3. **Thread-unsafe memoization** — In multi-threaded servers (Puma), `||=` is not atomic. Two threads can race and both execute the expensive computation. For truly expensive operations, use a `Mutex` or accept the occasional duplicate computation.

## Best Practices
1. **Scope memoization to the instance** — Use `@ivar ||=` so the cache is garbage-collected when the object is. Avoid `@@class_var` memoization in web applications.
2. **Bound your caches** — Any cache that accepts arbitrary keys should have a maximum size. Use an LRU strategy or TTL-based expiry to prevent unbounded growth.
3. **Document cache lifetimes** — Add a comment explaining how long the memoized value is valid and what invalidates it. Future maintainers will thank you.

## Summary
- `||=` is the standard Ruby memoization pattern but fails for `nil`/`false` values.
- Use `defined?(@ivar)` to memoize values that can be falsy.
- `Hash.new { |h, k| h[k] = compute(k) }` creates self-populating caches for parameterized methods.
- Unbounded caches cause memory bloat; use LRU eviction in long-running processes.
- Always consider the memory-vs-speed tradeoff before memoizing.

## Code Examples

**Three memoization patterns: basic ||= for non-nil values, defined?() guard for booleans, and Hash.new block for parameterized caching**

```ruby
class UserProfile
  def initialize(user_id)
    @user_id = user_id
  end

  # Simple memoization — safe when result is never nil/false
  def display_name
    @display_name ||= fetch_display_name
  end

  # Safe memoization for nil/false values
  def admin?
    return @is_admin if defined?(@is_admin)
    @is_admin = Permissions.admin?(@user_id)
  end

  # Parameterized memoization with Hash.new
  def permission(scope)
    @permissions ||= Hash.new { |h, s| h[s] = Permissions.check(@user_id, s) }
    @permissions[scope]
  end

  private

  def fetch_display_name
    User.find(@user_id).name
  end
end
```


## Resources

- [Ruby Memoization Techniques](https://docs.ruby-lang.org/en/4.0/Hash.html) — Ruby Hash documentation covering Hash.new with block for computed caches

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*