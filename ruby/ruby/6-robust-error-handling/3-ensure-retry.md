---
source_course: "ruby"
source_lesson: "ruby-ensure-retry"
---

# Ensure, Retry & Raise

## Introduction
Ruby's exception handling goes beyond simple `begin/rescue`. The `ensure` clause guarantees cleanup code runs no matter what, `retry` lets you re-attempt a failed operation, and `raise` within a rescue block lets you re-throw or wrap exceptions. Together, these tools let you write resilient code that cleans up after itself.

## Key Concepts
- **ensure**: A block that always executes after `begin/rescue`, whether an exception was raised or not. Used for resource cleanup.
- **retry**: A keyword that re-executes the `begin` block from the top. Used for transient failures like network timeouts.
- **else**: A clause that runs only when no exception was raised. Often overlooked but useful for separating the happy path from error handling.

## Real World Context
A background job that uploads files to cloud storage needs to close file handles even when uploads fail (ensure), retry on transient network errors with exponential backoff (retry), and re-raise permanent failures so the job framework can log them. Without these three tools, you end up with leaked resources and silent failures.

## Deep Dive

### The Full Begin/Rescue/Ensure Block

Ruby provides a complete structure for exception handling with `begin`, `rescue`, `else`, and `ensure`. Here is the full form:

```ruby
begin
  file = File.open("data.txt")
  process(file)
rescue IOError => e
  puts "Failed: \#{e.message}"
  raise  # Re-raise the same exception
else
  puts "Success!"  # Only runs if NO exception
ensure
  file&.close  # ALWAYS runs
end
```

The `else` clause runs only when no exception occurs, which keeps your happy-path logic separate from error recovery. The `ensure` clause always runs, making it the right place for cleanup.

### Ensure for Cleanup

`ensure` runs whether an exception occurred or not, making it ideal for releasing resources like database connections, file handles, or locks:

```ruby
def with_database
  conn = Database.connect
  yield conn
ensure
  conn&.disconnect  # Always disconnect
end
```

The safe navigation operator (`&.`) handles the case where `conn` is nil because `Database.connect` itself raised an error before assignment completed.

### Retry Logic

The `retry` keyword re-executes the `begin` block from the top. This is powerful for transient failures, but you must include a counter or timeout to avoid infinite loops:

```ruby
attempts = 0

begin
  attempts += 1
  response = make_api_call
rescue NetworkError
  if attempts < 3
    sleep(2 ** attempts)  # Exponential backoff
    retry
  else
    raise  # Give up after 3 attempts
  end
end
```

The exponential backoff (`2 ** attempts`) gives the remote service progressively more time to recover between retries. Without a retry limit, a persistent failure would loop forever.

### Inline Rescue

For simple cases, Ruby offers inline rescue. Use it sparingly because it rescues `StandardError` broadly and hides the specific error:

```ruby
# Returns default if any StandardError
result = risky_operation rescue default_value

# Better to be explicit about what you're catching
result = begin
  risky_operation
rescue ArgumentError
  default_value
end
```

The inline form is convenient for one-liners, but the explicit form is clearer about which errors you expect and what the fallback behavior is.

## Common Pitfalls
1. **Retry without a limit** — Forgetting to cap retries creates an infinite loop when the failure is permanent. Always track an attempt counter and re-raise after a maximum number of tries.
2. **Relying on inline rescue for important logic** — Inline rescue catches all `StandardError` subclasses indiscriminately and makes debugging hard. Reserve it for truly trivial defaults.

## Best Practices
1. **Always use ensure for resource cleanup** — File handles, database connections, and locks should be released in `ensure` blocks so they are freed regardless of whether an exception occurred.
2. **Use exponential backoff with retry** — Retrying immediately in a tight loop can overwhelm a struggling service. Exponential backoff gives the system time to recover.

## Summary
- `ensure` guarantees cleanup code runs whether or not an exception occurred, making it essential for resource management.
- `retry` re-executes the `begin` block and should always be paired with a maximum attempt counter to prevent infinite loops.
- Inline rescue is convenient but hides error details; prefer explicit `begin/rescue` for important logic.

## Code Examples

**Demonstrates a practical retry pattern with exponential backoff for HTTP requests.**

```ruby
# Retry with exponential backoff and ensure cleanup
def fetch_with_retry(url, max_attempts: 3)
  attempts = 0
  begin
    attempts += 1
    response = Net::HTTP.get_response(URI(url))
    raise "HTTP #{response.code}" unless response.is_a?(Net::HTTPSuccess)
    response.body
  rescue => e
    if attempts < max_attempts
      sleep(2 ** attempts)
      retry
    else
      raise
    end
  end
end
```

**Shows how ensure guarantees the file handle is closed even when an IOError occurs.**

```ruby
# Ensure block for guaranteed resource cleanup
def process_file(path)
  file = File.open(path, 'r')
  data = file.read
  # ... process data ...
  data
rescue IOError => e
  puts "Could not read file: #{e.message}"
  nil
ensure
  file&.close
  puts "File handle released"
end
```


## Resources

- [Exception Handling Syntax](https://docs.ruby-lang.org/en/4.0/syntax/exceptions_rdoc.html) — Official Ruby documentation covering begin, rescue, ensure, retry, and raise syntax.

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*