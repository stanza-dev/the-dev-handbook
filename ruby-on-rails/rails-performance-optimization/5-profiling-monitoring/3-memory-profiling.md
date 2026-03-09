---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-memory-profiling"
---

# Memory Profiling

## Introduction
Memory bloat slows down Ruby applications by increasing garbage collection time and can cause out-of-memory crashes. Profiling memory helps identify objects that allocate excessively and code paths that retain references unnecessarily.

## Key Concepts
- **Object Allocation**: Every Ruby operation creates objects on the heap. Excessive allocations increase GC pressure.
- **Memory Bloat**: When memory usage grows over time without being reclaimed — often caused by caching too much or retaining references.
- **GC (Garbage Collection)**: Ruby's automatic memory management that reclaims unused objects. More objects = more GC pauses.

## Real World Context
A Rails app processing CSV imports might allocate millions of string objects, causing 200ms GC pauses on every request. Profiling reveals the hot path, and switching to streaming CSV parsing eliminates the bloat.

## Deep Dive

### memory_profiler Gem

```ruby
# Gemfile
gem 'memory_profiler', group: :development
```

```ruby
report = MemoryProfiler.report do
  100.times { User.where(active: true).to_a }
end

report.pretty_print
# Shows: Total allocated: 45000 objects
# Allocated by gem, file, location, and class
```

### stackprof for CPU + Memory

```ruby
# Gemfile
gem 'stackprof', group: :development
```

```ruby
StackProf.run(mode: :object, out: 'tmp/stackprof-objects.dump') do
  # Code to profile
  User.all.map(&:full_name)
end

# View results
# bundle exec stackprof tmp/stackprof-objects.dump --text
```

### Reducing Allocations

```ruby
# BAD: Creates a new string on every call
def status_label
  "Status: #{status}".upcase
end

# GOOD: Use frozen string literals
# frozen_string_literal: true
STATUS_PREFIX = "Status: ".freeze

def status_label
  "#{STATUS_PREFIX}#{status.upcase}"
end
```

### Monitoring Memory in Production

```ruby
# Log memory usage per request
ActiveSupport::Notifications.subscribe('process_action.action_controller') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  memory_mb = `ps -o rss= -p #{Process.pid}`.to_i / 1024
  Rails.logger.info "Memory: #{memory_mb}MB after #{event.payload[:path]}"
end
```

## Common Pitfalls
1. **Loading all records with `.all.map`** — Use `.pluck` or `.find_each` to avoid instantiating thousands of ActiveRecord objects.
2. **Caching without size limits** — In-memory caches that grow unbounded will eventually consume all available memory.

## Best Practices
1. **Add `# frozen_string_literal: true`** — This directive at the top of every Ruby file freezes string literals, reducing allocations significantly.
2. **Use `find_each` for batch processing** — It loads records in batches, keeping memory usage constant regardless of total record count.

## Summary
- memory_profiler shows exactly which code allocates the most objects.
- stackprof profiles both CPU time and object allocations.
- Use frozen string literals and `.pluck` to reduce allocations.
- Monitor memory per process in production to detect bloat early.

## Code Examples

**Using memory_profiler to identify which code paths allocate the most objects**

```ruby
# Profile memory allocations in a code block
report = MemoryProfiler.report do
  users = User.where(active: true).to_a
  users.map { |u| "#{u.first_name} #{u.last_name}" }
end

report.pretty_print(to_file: 'tmp/memory_report.txt')
# Total allocated: 12500 objects (1.2 MB)
# Top allocations by location, gem, and class
```


## Resources

- [memory_profiler Gem](https://github.com/SamSaffron/memory_profiler) — Memory profiling gem for Ruby — shows allocations by gem, file, and location

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*