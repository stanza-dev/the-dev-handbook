---
source_course: "rails-api-development"
source_lesson: "rails-api-development-production-checklist"
---

# Production API Checklist

## Introduction
Before deploying an API to production, verify security, performance, and reliability. This checklist covers the essential items every production Rails API needs.

## Key Concepts
- **CORS**: Cross-Origin Resource Sharing headers that allow browser clients to call your API.
- **Request Logging**: Structured logs for debugging and monitoring API usage.
- **Health Check Endpoint**: A simple endpoint that monitoring services use to verify the API is running.
- **Security Headers**: HTTP headers that protect against common attacks.

## Real World Context
Production APIs face real-world challenges: malicious requests, high traffic, downtime, and debugging issues. The checklist below addresses the most common production issues.

## Deep Dive
### CORS Configuration

```ruby
# Gemfile
gem "rack-cors"

# config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins "https://myapp.com", "https://staging.myapp.com"
    resource "/api/*",
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete],
      expose: ["X-RateLimit-Limit", "X-RateLimit-Remaining"],
      max_age: 86400
  end
end
```

List specific origins. Never use `origins "*"` in production — it allows any website to call your API.

### Health Check

```ruby
# config/routes.rb
get "/health", to: proc { [200, {}, ['{"status":"ok"}']] }
```

A simple health check that monitoring services (Pingdom, UptimeRobot) can hit to verify the API is alive.

### Structured Logging

```ruby
# config/environments/production.rb
config.log_formatter = ::Logger::Formatter.new
config.logger = ActiveSupport::TaggedLogging.new(
  ActiveSupport::Logger.new(STDOUT)
)
```

Log to STDOUT for container-based deployments. Tagged logging adds context like request ID.

### Security Checklist

- Rate limiting on all endpoints (especially auth)
- CORS restricted to specific origins
- Input validation on all parameters
- SQL injection prevention (parameterized queries)
- No sensitive data in logs or error responses
- HTTPS enforced: `config.force_ssl = true`

## Common Pitfalls
1. **CORS with `origins "*"`** — Allows any website to call your API, enabling CSRF-like attacks.
2. **Missing rate limiting on auth endpoints** — Login endpoints without rate limiting enable brute-force attacks.

## Best Practices
1. **Deploy behind HTTPS** — Always. Use `config.force_ssl = true`.
2. **Add a health check endpoint** — Essential for load balancers and monitoring.

## Summary
- Configure CORS with specific origins for browser clients.
- Add a health check endpoint for monitoring.
- Use structured logging to STDOUT for container deployments.
- Enforce HTTPS, rate limiting, and input validation.
- Never expose sensitive data in logs or error responses.

## Code Examples

**Production CORS configuration with specific origins and exposed rate limit headers**

```ruby
# config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins "https://myapp.com"
    resource "/api/*",
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete],
      expose: ["X-RateLimit-Limit", "X-RateLimit-Remaining"]
  end
end
```


## Resources

- [Securing Rails Applications](https://guides.rubyonrails.org/security.html) — Rails security guide covering authentication, CSRF, injection, and more

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*