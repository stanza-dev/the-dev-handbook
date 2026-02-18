---
source_course: "rails-security-hardening"
source_lesson: "rails-security-security-headers"
---

# Security Headers

HTTP security headers provide defense-in-depth against various attacks.

## Default Rails Headers

Rails sets several headers by default:

```ruby
# X-Frame-Options: SAMEORIGIN (prevents clickjacking)
# X-XSS-Protection: 0 (disabled, CSP is better)
# X-Content-Type-Options: nosniff
# X-Permitted-Cross-Domain-Policies: none
```

## Configuring Headers

```ruby
# config/application.rb
config.action_dispatch.default_headers = {
  'X-Frame-Options' => 'DENY',
  'X-Content-Type-Options' => 'nosniff',
  'X-Permitted-Cross-Domain-Policies' => 'none',
  'Referrer-Policy' => 'strict-origin-when-cross-origin',
  'Permissions-Policy' => 'geolocation=(), microphone=(), camera=()'
}
```

## Strict Transport Security (HSTS)

```ruby
# config/environments/production.rb
config.force_ssl = true
config.ssl_options = {
  hsts: {
    expires: 1.year,
    subdomains: true,
    preload: true
  }
}
```

## Per-Controller Headers

```ruby
class AdminController < ApplicationController
  before_action :set_security_headers
  
  private
  
  def set_security_headers
    # More restrictive for admin area
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['Cache-Control'] = 'no-store'
  end
end
```

## Security Headers Middleware

```ruby
# app/middleware/security_headers.rb
class SecurityHeaders
  def initialize(app)
    @app = app
  end
  
  def call(env)
    status, headers, response = @app.call(env)
    
    headers['Permissions-Policy'] = [
      'geolocation=()',
      'microphone=()',
      'camera=()',
      'payment=(self)'
    ].join(', ')
    
    [status, headers, response]
  end
end

# config/application.rb
config.middleware.use SecurityHeaders
```

## Testing Headers

```ruby
class SecurityHeadersTest < ActionDispatch::IntegrationTest
  test 'security headers are set' do
    get root_url
    
    assert_equal 'DENY', response.headers['X-Frame-Options']
    assert_equal 'nosniff', response.headers['X-Content-Type-Options']
    assert response.headers['Content-Security-Policy'].present?
  end
end
```

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*