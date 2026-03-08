---
source_course: "rails-api-development"
source_lesson: "rails-api-development-error-handling"
---

# Structured Error Responses

## Introduction
APIs need consistent, machine-readable error responses. Every error — validation failure, not found, unauthorized, server error — should follow the same JSON structure so clients can handle errors generically.

## Key Concepts
- **Error Envelope**: A consistent JSON structure for all errors (e.g., `{ error: { message, status, code } }`).
- **`rescue_from`**: A Rails method that catches exceptions and renders custom responses.
- **Problem Details (RFC 7807)**: A standard for HTTP API error responses.
- **Validation Errors**: Errors returned when model validation fails.

## Real World Context
APIs that return inconsistent errors are painful to consume. Stripe, GitHub, and Twilio all use consistent error formats. A standard format means clients can handle all errors with a single error-handling function.

## Deep Dive
### Application-Level Error Handling

```ruby
class ApplicationController < ActionController::API
  rescue_from ActiveRecord::RecordNotFound, with: :not_found
  rescue_from ActiveRecord::RecordInvalid, with: :unprocessable_entity
  rescue_from ActionController::ParameterMissing, with: :bad_request

  private

  def not_found(exception)
    render json: {
      error: {
        status: 404,
        message: "Resource not found",
        detail: exception.message
      }
    }, status: :not_found
  end

  def unprocessable_entity(exception)
    render json: {
      error: {
        status: 422,
        message: "Validation failed",
        errors: exception.record.errors.full_messages
      }
    }, status: :unprocessable_entity
  end

  def bad_request(exception)
    render json: {
      error: {
        status: 400,
        message: "Bad request",
        detail: exception.message
      }
    }, status: :bad_request
  end
end
```

Every error follows the same shape: `status`, `message`, and additional details. Clients parse `error.message` for display and `error.status` for programmatic handling.

### Validation Error Details

```ruby
def create
  product = Product.new(product_params)
  if product.save
    render json: product, status: :created
  else
    render json: {
      error: {
        status: 422,
        message: "Validation failed",
        errors: product.errors.messages
      }
    }, status: :unprocessable_entity
  end
end
```

`errors.messages` returns a hash like `{ name: ["can't be blank"], price: ["must be greater than 0"] }`, letting clients show field-specific errors.

## Common Pitfalls
1. **Leaking stack traces in production** — Never include exception backtraces in API responses. Use generic messages for 500 errors.
2. **Inconsistent error formats** — If some endpoints return `{ message: "..." }` and others return `{ error: "..." }`, clients need special handling.

## Best Practices
1. **Handle errors at the application level** — `rescue_from` in ApplicationController catches errors from all endpoints.
2. **Return field-specific validation errors** — `errors.messages` lets clients highlight specific form fields.

## Summary
- Use a consistent error envelope for all API errors.
- `rescue_from` catches exceptions at the application level.
- Include field-specific validation errors for form handling.
- Never expose stack traces or internal details in production.

## Code Examples

**Application-level error handling ensuring consistent JSON error format across all endpoints**

```ruby
class ApplicationController < ActionController::API
  rescue_from ActiveRecord::RecordNotFound do |e|
    render json: {
      error: { status: 404, message: "Not found", detail: e.message }
    }, status: :not_found
  end

  rescue_from ActiveRecord::RecordInvalid do |e|
    render json: {
      error: { status: 422, message: "Validation failed",
               errors: e.record.errors.messages }
    }, status: :unprocessable_entity
  end
end
```


## Resources

- [Error Handling in Rails](https://guides.rubyonrails.org/action_controller_overview.html#rescue) — Rails guide on rescue_from and exception handling in controllers

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*