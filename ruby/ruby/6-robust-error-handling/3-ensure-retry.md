---
source_course: "ruby"
source_lesson: "ruby-ensure-retry"
---

# The Full Begin/Rescue/Ensure Block

```ruby
begin
  file = File.open("data.txt")
  process(file)
rescue IOError => e
  puts "Failed: #{e.message}"
  raise  # Re-raise the same exception
else
  puts "Success!"  # Only runs if NO exception
ensure
  file&.close  # ALWAYS runs
end
```

## Ensure for Cleanup

`ensure` runs whether an exception occurred or not:

```ruby
def with_database
  conn = Database.connect
  yield conn
ensure
  conn&.disconnect  # Always disconnect
end
```

## Retry Logic

The `retry` keyword re-executes the begin block:

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

## Inline Rescue

For simple cases, use inline rescue (use sparingly!):

```ruby
# Returns default if any error
result = risky_operation rescue default_value

# Better to be explicit about what you're catching
result = begin
  risky_operation
rescue ArgumentError
  default_value
end
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*