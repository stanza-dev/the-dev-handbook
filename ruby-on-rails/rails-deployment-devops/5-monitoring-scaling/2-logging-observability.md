---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-logging-observability"
---

# Structured Logging and Observability

## Introduction
Structured logging transforms Rails logs from human-readable text into machine-parseable JSON, making them searchable and analyzable at scale. This is essential for debugging production issues across multiple servers.

## Key Concepts
- **Structured Logging**: Logging in JSON format so log aggregation tools can index, search, and alert on specific fields.
- **Lograge**: A gem that replaces Rails' verbose multi-line request logs with single-line structured entries.
- **Request ID**: A unique identifier assigned to each request, allowing you to trace a single request across all log entries.

## Real World Context
With 3 web servers, finding logs for a specific failed request means searching through millions of lines. Structured logging with request IDs lets you filter by request_id in Datadog or CloudWatch and see the entire request lifecycle in seconds.

## Deep Dive

### Lograge Setup

```ruby
# Gemfile
gem 'lograge'

# config/initializers/lograge.rb
Rails.application.configure do
  config.lograge.enabled = true
  config.lograge.formatter = Lograge::Formatters::Json.new

  config.lograge.custom_payload do |controller|
    {
      user_id: controller.current_user&.id,
      request_id: controller.request.request_id
    }
  end
end
```

Output:
```json
{"method":"POST","path":"/orders","status":200,"duration":125.4,"db":23.1,"user_id":123}
```

### Production Logging Config

```ruby
# config/environments/production.rb
config.log_level = ENV.fetch('RAILS_LOG_LEVEL', 'info').to_sym
config.log_tags = [:request_id]
config.logger = ActiveSupport::TaggedLogging.new(
  ActiveSupport::Logger.new($stdout)
)
```

### Log Aggregation

Popular services:
- **Datadog Logs**: Full observability platform
- **Papertrail**: Simple log management
- **CloudWatch Logs**: AWS native

## Common Pitfalls
1. **Using :debug log level in production** — Generates enormous log volume and can slow down the application. Use :info or :warn.
2. **Not including request_id** — Without request IDs, you can't trace a single request across multiple log entries.

## Best Practices
1. **Use Lograge in production** — It reduces log volume by 90% while making logs machine-parseable.
2. **Log to stdout** — Container-based deployments (Docker, Kamal) expect logs on stdout for aggregation.

## Summary
- Lograge converts verbose Rails logs to single-line JSON entries.
- Include request_id and user_id in every log entry for tracing.
- Log to stdout for container-based deployments.
- Use a log aggregation service for searching across multiple servers.

## Code Examples

**Lograge configuration — converts Rails' multi-line logs into single-line JSON with custom fields**

```ruby
# config/initializers/lograge.rb
Rails.application.configure do
  config.lograge.enabled = true
  config.lograge.formatter = Lograge::Formatters::Json.new
  config.lograge.custom_payload do |controller|
    {
      user_id: controller.current_user&.id,
      request_id: controller.request.request_id
    }
  end
end

# Output: {"method":"GET","path":"/orders","status":200,
#          "duration":45.2,"user_id":42,"request_id":"abc-123"}
```


## Resources

- [Lograge Gem](https://github.com/roidrage/lograge) — Gem that tames Rails logging — converts verbose logs to structured single-line entries

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*