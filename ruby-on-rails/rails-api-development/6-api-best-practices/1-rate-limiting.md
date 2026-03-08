---
source_course: "rails-api-development"
source_lesson: "rails-api-development-rate-limiting"
---

# Rate Limiting

## Introduction
Rate limiting protects your API from abuse, DDoS attacks, and runaway scripts. Rails 8 introduced a built-in `rate_limit` method that makes basic rate limiting trivial to implement.

## Key Concepts
- **`rate_limit`**: Rails 8's built-in controller method for declaring rate limits.
- **Rack::Attack**: A middleware gem for more advanced rate limiting patterns.
- **Rate Window**: The time period over which requests are counted (e.g., 60 requests per minute).
- **Throttling**: Slowing down or rejecting requests that exceed the limit.

## Real World Context
Without rate limiting, a single client can overwhelm your API, degrading performance for everyone. Rate limiting is standard practice: GitHub allows 5,000 requests per hour, Stripe allows 100 requests per second, and Twitter uses tiered limits.

## Deep Dive
### Rails 8 Built-in rate_limit

Rails 8 added a `rate_limit` method directly in controllers:

```ruby
class Api::V1::ProductsController < ApplicationController
  rate_limit to: 60, within: 1.minute, by: -> { request.remote_ip }

  def index
    render json: Product.all
  end
end
```

This limits each IP address to 60 requests per minute. When exceeded, Rails returns a `429 Too Many Requests` response automatically. No gem needed.

You can set different limits per action:

```ruby
class Api::V1::AuthController < ApplicationController
  rate_limit to: 5, within: 1.minute, only: :create,
             by: -> { request.remote_ip }

  def create
    # Login endpoint — stricter limit to prevent brute force
  end
end
```

The login endpoint gets a stricter limit (5 per minute) to prevent brute-force attacks.

### Rack::Attack for Advanced Patterns

For more complex rate limiting, use Rack::Attack:

```ruby
# Gemfile
gem "rack-attack"

# config/initializers/rack_attack.rb
Rack::Attack.throttle("api/ip", limit: 300, period: 5.minutes) do |req|
  req.ip if req.path.start_with?("/api/")
end

Rack::Attack.throttle("api/token", limit: 100, period: 1.minute) do |req|
  req.env["HTTP_AUTHORIZATION"]&.remove("Bearer ") if req.path.start_with?("/api/")
end

# Custom response for throttled requests
Rack::Attack.throttled_responder = lambda do |request|
  [429, { "Content-Type" => "application/json" },
   [{ error: "Rate limit exceeded. Retry after #{request.env['rack.attack.match_data'][:period]} seconds" }.to_json]]
end
```

Rack::Attack supports multiple throttle rules, blocklists, safelists, and per-token limits.

### Rate Limit Headers

Good APIs communicate rate limit status in response headers:

```ruby
after_action :set_rate_limit_headers

def set_rate_limit_headers
  response.headers["X-RateLimit-Limit"] = "60"
  response.headers["X-RateLimit-Remaining"] = remaining_requests.to_s
  response.headers["X-RateLimit-Reset"] = reset_time.to_i.to_s
end
```

## Common Pitfalls
1. **Rate limiting only by IP** — Behind load balancers, all requests may share an IP. Limit by API token when possible.
2. **No rate limit headers** — Clients can't implement backoff without knowing their limit status.

## Best Practices
1. **Use Rails 8 `rate_limit` for simple cases** — No gem needed for basic per-IP or per-token limits.
2. **Use Rack::Attack for complex rules** — Multiple tiers, blocklists, and custom responses.

## Summary
- Rails 8's `rate_limit` provides built-in rate limiting with no additional gems.
- Rack::Attack handles advanced patterns like per-token limits and blocklists.
- Return 429 Too Many Requests when limits are exceeded.
- Include rate limit headers so clients can implement backoff.
- Set stricter limits on sensitive endpoints like login.

## Code Examples

**Rails 8's built-in rate_limit — declare limits directly in controllers without any gems**

```ruby
# Rails 8 built-in rate limiting — no gem needed
class Api::V1::ProductsController < ApplicationController
  rate_limit to: 60, within: 1.minute,
             by: -> { request.remote_ip }
  # 60 requests per minute per IP
  # Returns 429 Too Many Requests when exceeded
end

class Api::V1::AuthController < ApplicationController
  rate_limit to: 5, within: 1.minute, only: :create,
             by: -> { request.remote_ip }
  # Stricter limit on login to prevent brute force
end
```


## Resources

- [Action Controller Rate Limiting](https://api.rubyonrails.org/classes/ActionController/RateLimiting/ClassMethods.html) — Rails API docs for the built-in rate_limit method

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*