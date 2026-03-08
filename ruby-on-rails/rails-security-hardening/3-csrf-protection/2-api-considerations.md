---
source_course: "rails-security-hardening"
source_lesson: "rails-security-csrf-api-considerations"
---

# CSRF API Considerations

## Introduction
APIs typically do not use browser cookies for authentication, which means they are not vulnerable to CSRF in the same way as web pages. Understanding when to skip CSRF protection and what to replace it with is critical for mixed applications.

## Key Concepts
- **ActionController::API**: A lightweight Rails controller base class that excludes CSRF protection, designed for API-only applications.
- **Token Authentication**: Using bearer tokens in HTTP headers instead of cookies, which are not automatically included by the browser.
- **Webhook Signature**: A cryptographic hash used to verify that a webhook payload was sent by the expected service.

## Real World Context
Most modern Rails applications serve both web pages and API endpoints. The web pages need CSRF protection, while API endpoints use token-based authentication. Webhooks from third-party services need their own verification mechanism.

## Deep Dive

### API-Only Applications

API-only apps inherit from `ActionController::API`, which does not include CSRF protection:

```ruby
class ApplicationController < ActionController::API
  # No CSRF protection — use token-based auth instead
end
```

Since API clients send authentication tokens in headers rather than cookies, CSRF attacks cannot work against them.

### Mixed Applications

For apps with both web views and API endpoints, create a separate base controller:

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
end

class Api::BaseController < ApplicationController
  skip_before_action :verify_authenticity_token

  before_action :authenticate_api_token

  private

  def authenticate_api_token
    token = request.headers['Authorization']&.split(' ')&.last
    @current_user = User.find_by_api_token(token)

    render json: { error: 'Unauthorized' }, status: :unauthorized unless @current_user
  end
end
```

The API controller skips CSRF verification but replaces it with token authentication, maintaining security through a different mechanism.

### Webhook Endpoints

Webhooks from third-party services need signature verification instead of CSRF tokens:

```ruby
class WebhooksController < ApplicationController
  skip_before_action :verify_authenticity_token, only: :receive

  before_action :verify_webhook_signature, only: :receive

  def receive
    # Process webhook payload
  end

  private

  def verify_webhook_signature
    signature = request.headers['X-Webhook-Signature']
    payload = request.raw_post

    unless valid_signature?(payload, signature)
      render json: { error: 'Invalid signature' }, status: :forbidden
    end
  end
end
```

Webhook signatures prove that the request came from the expected service, not from an attacker.

### Null Session Strategy

An alternative to raising exceptions on invalid tokens:

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :null_session
end
```

This clears the session instead of raising an error, which can be useful when you want to allow unauthenticated access as a fallback.

## Common Pitfalls
1. **Skipping CSRF globally for API convenience** — Always limit CSRF skipping to specific controllers or actions that have alternative authentication.
2. **Not verifying webhook signatures** — Skipping CSRF for webhooks without signature verification lets anyone send fake webhook payloads.

## Best Practices
1. **Separate web and API controller hierarchies** — Use `ApplicationController` for web and `Api::BaseController` for APIs, each with appropriate authentication.
2. **Always replace CSRF with something** — When you skip CSRF protection, you must have token auth, signature verification, or another mechanism in its place.

## Summary
- API-only applications use `ActionController::API` which excludes CSRF.
- Mixed applications need separate controller hierarchies for web and API.
- Always replace CSRF protection with an alternative authentication mechanism.
- Webhook endpoints should verify payload signatures from the sending service.

## Code Examples

**Controller hierarchy pattern for mixed applications — web controllers use CSRF, API controllers use token authentication**

```ruby
# Mixed app: separate controller hierarchies
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception  # Web pages
end

class Api::BaseController < ApplicationController
  skip_before_action :verify_authenticity_token
  before_action :authenticate_api_token  # Token auth instead
end
```


## Resources

- [Rails API-Only Applications](https://guides.rubyonrails.org/api_app.html) — Official Rails guide on building API-only applications

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*