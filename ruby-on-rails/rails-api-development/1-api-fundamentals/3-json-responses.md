---
source_course: "rails-api-development"
source_lesson: "rails-api-development-json-responses"
---

# Structuring JSON Responses

## Introduction
Consistent JSON response formatting is crucial for API usability. Rails provides several approaches for controlling what data is included in responses, from simple `as_json` overrides to Jbuilder templates.

## Key Concepts
- **`as_json`**: A method you override on models to control their default JSON representation.
- **`render json:`**: Controller method that converts objects to JSON. Calls `as_json` on the object.
- **Envelope Pattern**: Wrapping response data in a top-level key (e.g., `{ "data": [...] }`).
- **Jbuilder**: A Rails DSL for building JSON responses with fine-grained control.

## Real World Context
API clients need predictable response shapes. Every endpoint should follow the same structure. Inconsistent responses make APIs hard to consume and lead to fragile client code.

## Deep Dive
### Controlling Output with `as_json`

Override `as_json` on your model to set the default JSON shape:

```ruby
class Product < ApplicationRecord
  def as_json(options = {})
    super(options.merge(
      only: [:id, :name, :price, :description],
      methods: [:formatted_price],
      include: { category: { only: [:id, :name] } }
    ))
  end

  def formatted_price
    "$#{price.to_f / 100}"
  end
end
```

Now `render json: @product` automatically uses this shape. The `only` option whitelists fields, `methods` adds computed values, and `include` nests associations.

### Using Jbuilder for Complex Responses

For more control, use Jbuilder templates:

```ruby
# app/views/api/v1/products/index.json.jbuilder
json.data @products do |product|
  json.id product.id
  json.name product.name
  json.price product.formatted_price
  json.category do
    json.id product.category.id
    json.name product.category.name
  end
  json.links do
    json.self api_v1_product_url(product)
  end
end
json.meta do
  json.total @products.count
end
```

Jbuilder templates live alongside HTML views and give you a clean DSL for building nested JSON structures. The controller just sets instance variables:

```ruby
def index
  @products = Product.includes(:category).all
  # Renders index.json.jbuilder automatically
end
```

### Consistent Error Responses

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  rescue_from ActiveRecord::RecordNotFound, with: :not_found
  rescue_from ActionController::ParameterMissing, with: :bad_request

  private

  def not_found(exception)
    render json: { error: { message: exception.message, status: 404 } }, status: :not_found
  end

  def bad_request(exception)
    render json: { error: { message: exception.message, status: 400 } }, status: :bad_request
  end
end
```

Using `rescue_from` at the application level ensures every 404 and 400 follows the same format.

## Common Pitfalls
1. **Exposing sensitive fields** — Without `only` or `except`, `render json: @user` exposes everything including `password_digest`, `email`, and internal fields.
2. **Inconsistent response shapes** — If some endpoints wrap in `{ data: ... }` and others don't, clients need special handling for each endpoint.

## Best Practices
1. **Always whitelist fields** — Use `only:` in `as_json` or Jbuilder to explicitly list fields. Never render full model attributes.
2. **Standardize error format** — Every error response should follow the same structure across all endpoints.

## Summary
- Override `as_json` for simple JSON customization with field whitelisting.
- Use Jbuilder templates for complex nested responses with computed fields.
- Use `rescue_from` for consistent error response formatting.
- Always whitelist fields to avoid exposing sensitive data.
- Wrap collections in an envelope (`data`, `meta`) for extensibility.

## Code Examples

**Application-level error handling ensuring consistent JSON error format across all endpoints**

```ruby
# Consistent error handling in the application controller
class ApplicationController < ActionController::API
  rescue_from ActiveRecord::RecordNotFound, with: :not_found
  rescue_from ActionController::ParameterMissing, with: :bad_request

  private

  def not_found(e)
    render json: { error: { message: e.message, status: 404 } },
           status: :not_found
  end

  def bad_request(e)
    render json: { error: { message: e.message, status: 400 } },
           status: :bad_request
  end
end
```


## Resources

- [Rails API App Guide](https://guides.rubyonrails.org/api_app.html) — Guide covering JSON rendering, as_json, and API response patterns

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*