---
source_course: "rails-security-hardening"
source_lesson: "rails-security-csrf-api-considerations"
---

# CSRF API Considerations

APIs typically don't use CSRF tokens. Here's how to handle this safely.

## API-Only Applications

API-only apps inherit from `ActionController::API`, which doesn't include CSRF:

```ruby
class ApplicationController < ActionController::API
  # No CSRF protection - use token-based auth instead
end
```

## Mixed Applications

For apps with both web views and API endpoints:

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
end

class Api::BaseController < ApplicationController
  # Skip CSRF for API requests
  skip_before_action :verify_authenticity_token
  
  # Use token authentication instead
  before_action :authenticate_api_token
  
  private
  
  def authenticate_api_token
    token = request.headers['Authorization']&.split(' ')&.last
    @current_user = User.find_by_api_token(token)
    
    render json: { error: 'Unauthorized' }, status: :unauthorized unless @current_user
  end
end
```

## Per-Action CSRF Skipping

```ruby
class WebhooksController < ApplicationController
  # Skip only for webhook endpoint
  skip_before_action :verify_authenticity_token, only: :receive
  
  # Use webhook signature verification instead
  before_action :verify_webhook_signature, only: :receive
  
  def receive
    # Process webhook
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

## Null Session Strategy

Alternative to raising exceptions:

```ruby
class ApplicationController < ActionController::Base
  # Clears session instead of raising error
  protect_from_forgery with: :null_session
end
```

Useful when you want to allow some unauthenticated API access.

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*