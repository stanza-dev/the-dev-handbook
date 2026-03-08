---
source_course: "rails-api-development"
source_lesson: "rails-api-development-serializer-patterns"
---

# Customizing JSON with as_json

## Introduction
Every ActiveRecord model can be rendered as JSON with `render json: @model`. By default, this exposes all database columns. The `as_json` method lets you control exactly which fields appear in your API responses.

## Key Concepts
- **`as_json`**: Method called when converting an object to JSON. Override it on models to customize the output.
- **`only` / `except`**: Options to whitelist or blacklist specific attributes.
- **`methods`**: Option to include computed method results in the JSON output.
- **`include`**: Option to nest associated models in the response.

## Real World Context
Without customizing `as_json`, your API will expose internal fields like `password_digest`, `created_at`, `updated_at`, and foreign keys. This is both a security risk and poor API design. Every production API needs explicit control over response shapes.

## Deep Dive
### Basic Customization

```ruby
class Product < ApplicationRecord
  belongs_to :category
  has_many :reviews

  def as_json(options = {})
    super(options.merge(
      only: [:id, :name, :price, :description, :stock_count],
      methods: [:formatted_price, :average_rating],
      include: {
        category: { only: [:id, :name] }
      }
    ))
  end

  def formatted_price
    "$#{'%.2f' % (price.to_f / 100)}"
  end

  def average_rating
    reviews.average(:rating)&.round(1) || 0.0
  end
end
```

Now `render json: @product` outputs only the specified fields plus computed values. Internal columns like `created_at` and `category_id` are excluded.

### Controller-Level Customization

You can also customize at the controller level:

```ruby
def show
  product = Product.find(params[:id])
  render json: product.as_json(
    only: [:id, :name, :price],
    include: {
      reviews: {
        only: [:id, :body, :rating],
        include: { author: { only: [:id, :name] } }
      }
    }
  )
end
```

Controller-level customization overrides the model default, giving you endpoint-specific shapes.

### Collection Rendering

For collections, each item uses the same `as_json` rules:

```ruby
def index
  products = Product.includes(:category).all
  render json: { data: products, meta: { total: products.count } }
end
```

Each product in the array is serialized using the model's `as_json` method. The `includes(:category)` prevents N+1 queries.

## Common Pitfalls
1. **N+1 queries with `include`** — Using `include:` in `as_json` without `includes()` in the query triggers a database query per record.
2. **Forgetting to whitelist fields** — Without `only:`, all columns are exposed.

## Best Practices
1. **Always use `only:` to whitelist** — Explicit field listing prevents accidental exposure of sensitive data.
2. **Eager load associations** — Always use `.includes()` when your JSON includes associations.

## Summary
- Override `as_json` on models to control default JSON representation.
- Use `only:` to whitelist fields, `methods:` for computed values, `include:` for associations.
- Controller-level `as_json` options override model defaults.
- Always eager load associations to prevent N+1 queries.

## Code Examples

**Safe user serialization that whitelists only public fields and adds computed values**

```ruby
class User < ApplicationRecord
  def as_json(options = {})
    super(options.merge(
      only: [:id, :name, :email],
      methods: [:avatar_url, :posts_count]
    ))
    # Never exposes: password_digest, api_token,
    # remember_digest, admin flag, etc.
  end
end
```


## Resources

- [as_json API Documentation](https://api.rubyonrails.org/classes/ActiveModel/Serializers/JSON.html) — ActiveModel as_json API reference

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*