---
source_course: "rails-api-development"
source_lesson: "rails-api-development-rate-limiting"
---

# Rate Limiting

Protect your API from abuse by limiting how many requests a client can make.

## Using Rack::Attack

```ruby
# Gemfile
gem 'rack-attack'

# config/initializers/rack_attack.rb
class Rack::Attack
  # Throttle all requests by IP
  throttle('requests/ip', limit: 300, period: 5.minutes) do |req|
    req.ip
  end

  # Throttle API requests by token
  throttle('api/token', limit: 100, period: 1.minute) do |req|
    if req.path.start_with?('/api/')
      req.env['HTTP_AUTHORIZATION']&.split(' ')&.last
    end
  end

  # Throttle login attempts
  throttle('logins/ip', limit: 5, period: 20.seconds) do |req|
    if req.path == '/api/v1/auth' && req.post?
      req.ip
    end
  end

  # Throttle login by email
  throttle('logins/email', limit: 5, period: 1.minute) do |req|
    if req.path == '/api/v1/auth' && req.post?
      req.params['email'].to_s.downcase.gsub(/\s+/, '')
    end
  end

  # Block suspicious requests
  blocklist('block bad IPs') do |req|
    Rack::Attack::Fail2Ban.filter("aggressive-#{req.ip}", maxretry: 10, findtime: 1.minute, bantime: 1.hour) do
      req.path.include?('/etc/passwd') ||
      req.path.include?('.php') ||
      req.path.include?('wp-admin')
    end
  end

  # Allowlist (skip throttling)
  safelist('allow from internal') do |req|
    req.ip == '127.0.0.1' || req.ip == '::1'
  end
end

# config/application.rb
config.middleware.use Rack::Attack
```

## Rate Limit Headers

```ruby
# config/initializers/rack_attack.rb
Rack::Attack.throttled_responder = lambda do |request|
  match_data = request.env['rack.attack.match_data']
  now = Time.now.utc

  headers = {
    'Content-Type' => 'application/json',
    'X-RateLimit-Limit' => match_data[:limit].to_s,
    'X-RateLimit-Remaining' => '0',
    'X-RateLimit-Reset' => (now + (match_data[:period] - now.to_i % match_data[:period])).to_i.to_s,
    'Retry-After' => (match_data[:period] - now.to_i % match_data[:period]).to_s
  }

  body = {
    error: 'Rate limit exceeded',
    retry_after: headers['Retry-After'].to_i
  }.to_json

  [429, headers, [body]]
end
```

## Adding Rate Limit Info to All Responses

```ruby
class Api::V1::BaseController < ApplicationController
  after_action :set_rate_limit_headers

  private

  def set_rate_limit_headers
    return unless request.env['rack.attack.throttle_data']

    data = request.env['rack.attack.throttle_data']['api/token']
    return unless data

    response.headers['X-RateLimit-Limit'] = data[:limit].to_s
    response.headers['X-RateLimit-Remaining'] = (data[:limit] - data[:count]).to_s
    response.headers['X-RateLimit-Reset'] = data[:epoch_time].to_s
  end
end
```

## Client Handling

Clients should check these headers:

```javascript
const response = await fetch('/api/v1/articles');

if (response.status === 429) {
  const retryAfter = response.headers.get('Retry-After');
  console.log(`Rate limited. Retry in ${retryAfter} seconds`);
}

const remaining = response.headers.get('X-RateLimit-Remaining');
console.log(`${remaining} requests remaining`);
```

## Resources

- [Rack::Attack](https://github.com/rack/rack-attack) — Rate limiting and blocking for Rack apps

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*