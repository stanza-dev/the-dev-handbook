---
source_course: "rails-api-development"
source_lesson: "rails-api-development-error-handling"
---

# Comprehensive Error Handling

Produce consistent, helpful error responses across your entire API.

## Global Exception Handling

```ruby
class Api::V1::BaseController < ApplicationController
  rescue_from StandardError, with: :handle_standard_error
  rescue_from ActiveRecord::RecordNotFound, with: :handle_not_found
  rescue_from ActiveRecord::RecordInvalid, with: :handle_validation_error
  rescue_from ActionController::ParameterMissing, with: :handle_parameter_missing
  rescue_from AuthenticationError, with: :handle_authentication_error
  rescue_from AuthorizationError, with: :handle_authorization_error

  private

  def handle_not_found(exception)
    render json: {
      error: {
        code: 'not_found',
        message: "Resource not found",
        details: exception.message
      }
    }, status: :not_found
  end

  def handle_validation_error(exception)
    render json: {
      error: {
        code: 'validation_error',
        message: 'Validation failed',
        details: format_validation_errors(exception.record)
      }
    }, status: :unprocessable_entity
  end

  def handle_parameter_missing(exception)
    render json: {
      error: {
        code: 'parameter_missing',
        message: "Required parameter missing: #{exception.param}"
      }
    }, status: :bad_request
  end

  def handle_authentication_error(exception)
    render json: {
      error: {
        code: 'unauthorized',
        message: exception.message || 'Authentication required'
      }
    }, status: :unauthorized
  end

  def handle_authorization_error(exception)
    render json: {
      error: {
        code: 'forbidden',
        message: exception.message || 'Access denied'
      }
    }, status: :forbidden
  end

  def handle_standard_error(exception)
    # Log the error
    Rails.logger.error(exception.message)
    Rails.logger.error(exception.backtrace.join("\n"))

    # Report to error tracking service
    Sentry.capture_exception(exception) if defined?(Sentry)

    # Don't expose internal errors in production
    if Rails.env.production?
      render json: {
        error: {
          code: 'internal_error',
          message: 'An unexpected error occurred'
        }
      }, status: :internal_server_error
    else
      render json: {
        error: {
          code: 'internal_error',
          message: exception.message,
          backtrace: exception.backtrace.first(10)
        }
      }, status: :internal_server_error
    end
  end

  def format_validation_errors(record)
    record.errors.map do |error|
      {
        field: error.attribute.to_s,
        message: error.message,
        code: error.type.to_s
      }
    end
  end
end
```

## Custom Error Classes

```ruby
# app/errors/api_error.rb
class ApiError < StandardError
  attr_reader :code, :status

  def initialize(message, code: nil, status: :bad_request)
    super(message)
    @code = code || 'api_error'
    @status = status
  end
end

class AuthenticationError < ApiError
  def initialize(message = 'Authentication required')
    super(message, code: 'unauthorized', status: :unauthorized)
  end
end

class AuthorizationError < ApiError
  def initialize(message = 'Access denied')
    super(message, code: 'forbidden', status: :forbidden)
  end
end

class RateLimitError < ApiError
  def initialize
    super('Rate limit exceeded', code: 'rate_limited', status: :too_many_requests)
  end
end
```

## Error Response Schema

```yaml
# Always return errors in this format:
{
  "error": {
    "code": "validation_error",
    "message": "Human readable message",
    "details": [
      {
        "field": "email",
        "message": "is invalid",
        "code": "invalid"
      }
    ],
    "request_id": "abc-123"  # For debugging
  }
}
```

## Request ID Tracking

```ruby
class Api::V1::BaseController < ApplicationController
  before_action :set_request_id

  private

  def set_request_id
    @request_id = request.headers['X-Request-Id'] || SecureRandom.uuid
    response.headers['X-Request-Id'] = @request_id
  end
end
```

## Resources

- [Error Reporting in Rails](https://guides.rubyonrails.org/error_reporting.html) — Rails error reporting guide

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*