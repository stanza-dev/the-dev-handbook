---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-logging-observability"
---

# Logging Observability

Structured logging makes logs searchable and analyzable.

## Structured JSON Logging

```ruby
# config/environments/production.rb
config.log_level = ENV.fetch('RAILS_LOG_LEVEL', 'info').to_sym
config.log_tags = [:request_id]

# Use JSON formatter
config.logger = ActiveSupport::TaggedLogging.new(
  ActiveSupport::Logger.new($stdout)
)
```

## Lograge for Clean Logs

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
      request_id: controller.request.request_id,
      ip: controller.request.remote_ip
    }
  end
  
  config.lograge.custom_options = lambda do |event|
    {
      params: event.payload[:params].except('controller', 'action'),
      exception: event.payload[:exception]&.first,
      exception_message: event.payload[:exception_object]&.message
    }
  end
end
```

Output:
```json
{"method":"POST","path":"/orders","format":"html","controller":"OrdersController","action":"create","status":200,"duration":125.4,"view":45.2,"db":23.1,"user_id":123}
```

## Log Aggregation Services

- **Papertrail**: Simple log management
- **Datadog Logs**: Full observability platform
- **Elastic Stack**: Self-hosted option
- **CloudWatch Logs**: AWS native

```ruby
# Example: Datadog logging
# config/initializers/datadog.rb
Datadog.configure do |c|
  c.service = 'my-rails-app'
  c.env = Rails.env
  c.version = ENV['GIT_SHA']
  c.tracing.enabled = true
end
```

## Distributed Tracing

```ruby
# Trace requests across services
ActiveSupport::Notifications.instrument('api.external_call', service: 'payment') do
  PaymentGateway.charge(order)
end
```

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*