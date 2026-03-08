---
source_course: "rails-api-development"
source_lesson: "rails-api-development-serializer-libs"
---

# Serializer Libraries

## Introduction
For complex APIs, dedicated serializer libraries provide more structure than `as_json` or Jbuilder. They centralize serialization logic in dedicated classes, support JSON:API format, and offer performance optimizations.

## Key Concepts
- **Serializer Class**: A dedicated class responsible for converting a model to JSON.
- **JSON:API Format**: A specification for consistent API response structure with `data`, `attributes`, `relationships`.
- **`jsonapi-serializer`**: The most popular Rails serializer gem (formerly `fast_jsonapi`).
- **Field Selection**: Letting API clients request only specific fields (`?fields[product]=name,price`).

## Real World Context
As APIs grow, serialization logic scattered across models and controllers becomes unmaintainable. Serializer classes centralize this logic, making it testable and consistent. The JSON:API specification provides a standard that API clients can consume generically.

## Deep Dive
### Using jsonapi-serializer

```ruby
# Gemfile
gem "jsonapi-serializer"
```

```ruby
# app/serializers/product_serializer.rb
class ProductSerializer
  include JSONAPI::Serializer

  attributes :name, :description, :price
  attribute :formatted_price do |product|
    "$#{'%.2f' % (product.price.to_f / 100)}"
  end

  belongs_to :category
  has_many :reviews
end
```

Usage in controllers:

```ruby
def index
  products = Product.includes(:category).all
  render json: ProductSerializer.new(products).serializable_hash.to_json
end

def show
  product = Product.find(params[:id])
  options = { include: [:category, :reviews] }
  render json: ProductSerializer.new(product, options).serializable_hash.to_json
end
```

The output follows JSON:API format with `data`, `id`, `type`, `attributes`, and `relationships` keys.

### Choosing an Approach

| Approach | Best For | Drawbacks |
|----------|----------|-----------|
| `as_json` | Simple APIs, quick prototypes | Logic scattered on models |
| Jbuilder | Complex response shapes, conditional fields | Templates can get large |
| jsonapi-serializer | JSON:API compliance, large APIs | Additional dependency, JSON:API overhead |

## Common Pitfalls
1. **Mixing serialization approaches** — Pick one strategy and use it consistently across all endpoints.
2. **Over-serializing** — Including every association by default. Let clients request what they need.

## Best Practices
1. **Start simple, evolve as needed** — `as_json` for small APIs, Jbuilder for medium, serializer library for large.
2. **Support sparse fieldsets** — Let clients specify which fields they want to reduce payload size.

## Summary
- Serializer libraries centralize JSON transformation in dedicated classes.
- `jsonapi-serializer` provides JSON:API-compliant output with relationships.
- Choose your approach based on API size: as_json → Jbuilder → serializer library.
- Support sparse fieldsets for flexible client consumption.

## Code Examples

**A jsonapi-serializer class with attributes, computed fields, and relationships**

```ruby
class ProductSerializer
  include JSONAPI::Serializer

  attributes :name, :description, :price

  attribute :formatted_price do |product|
    "$#{'%.2f' % (product.price.to_f / 100)}"
  end

  belongs_to :category
  has_many :reviews
end

# Usage: ProductSerializer.new(@product).serializable_hash
```


## Resources

- [jsonapi-serializer](https://github.com/jsonapi-serializer/jsonapi-serializer) — The jsonapi-serializer gem for JSON:API-compliant serialization

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*