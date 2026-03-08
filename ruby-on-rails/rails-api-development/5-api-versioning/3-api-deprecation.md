---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-deprecation"
---

# API Deprecation Strategies

## Introduction
As your API evolves, you need a clear strategy for deprecating old endpoints and communicating changes to consumers. Rails provides tools and conventions that make this process smooth.

## Key Concepts
- **Deprecation Header**: The `Sunset` and `Deprecation` HTTP headers signal to clients that an endpoint is being retired.
- **Versioned Deprecation**: Marking entire API versions as deprecated while keeping them functional during a transition period.
- **Deprecation Timeline**: A published schedule giving consumers time to migrate.

## Real World Context
Breaking API changes without warning cause production outages for your consumers. Companies like Stripe and GitHub maintain deprecation policies that give developers months of notice before removing endpoints.

## Deep Dive

### Using HTTP Headers for Deprecation

Rails makes it easy to add deprecation headers in your controllers:

```ruby
class Api::V1::UsersController < ApplicationController
  before_action :add_deprecation_headers

  private

  def add_deprecation_headers
    response.headers['Deprecation'] = 'true'
    response.headers['Sunset'] = 'Sat, 01 Jan 2027 00:00:00 GMT'
    response.headers['Link'] = '<https://api.example.com/v2/users>; rel="successor-version"'
  end
end
```

The `Sunset` header tells clients the exact date the endpoint will stop working. The `Link` header points them to the replacement.

### Rails Deprecation Warnings

For internal deprecation tracking, use Rails' built-in deprecation system:

```ruby
ActiveSupport::Deprecation.warn(
  "API v1 is deprecated. Please migrate to v2 by January 2027."
)
```

This logs warnings and can be configured to report to your monitoring system.

### Deprecation Middleware

For a systematic approach, create middleware that handles deprecation across all v1 endpoints:

```ruby
class Api::DeprecationMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    status, headers, response = @app.call(env)
    if env['PATH_INFO'].start_with?('/api/v1')
      headers['Deprecation'] = 'true'
      headers['Sunset'] = 'Sat, 01 Jan 2027 00:00:00 GMT'
    end
    [status, headers, response]
  end
end
```

## Common Pitfalls
1. **Removing endpoints without notice** — Always add deprecation headers weeks or months before removing an endpoint. Monitor usage to ensure consumers have migrated.
2. **Not providing migration guides** — Deprecation headers alone aren't enough. Document what changed and how to migrate in your API changelog.

## Best Practices
1. **Use the Sunset header standard (RFC 8594)** — It's the industry standard for communicating endpoint retirement dates.
2. **Monitor deprecated endpoint usage** — Track how many requests still hit deprecated endpoints before removing them.
3. **Version your deprecation policy** — Publish a clear policy (e.g., "deprecated endpoints are removed after 6 months").

## Summary
- Use `Deprecation` and `Sunset` HTTP headers to signal upcoming removals.
- Provide `Link` headers pointing to replacement endpoints.
- Monitor usage of deprecated endpoints before removal.
- Publish clear deprecation timelines and migration guides.

## Code Examples

**Configuring Rails deprecation notifications to track deprecated endpoint usage in your monitoring system**

```ruby
# config/application.rb
# Configure deprecation behavior per environment
config.active_support.deprecation = :notify

# Subscribe to deprecation events for monitoring
ActiveSupport::Notifications.subscribe('deprecation.rails') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  Rails.logger.warn(event.payload[:message])
  # Send to your monitoring service
  StatsD.increment('api.deprecation_warning')
end
```


## Resources

- [RFC 8594 - The Sunset HTTP Header Field](https://datatracker.ietf.org/doc/html/rfc8594) — The official standard for the Sunset header used to signal API deprecation

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*