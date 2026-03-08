---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-standards"
---

# Building JSON with Jbuilder

## Introduction
Jbuilder is a Rails-native templating DSL for building JSON responses. Instead of overriding `as_json` on models, you create `.json.jbuilder` view templates that give you full control over response structure with a clean Ruby syntax.

## Key Concepts
- **Jbuilder Template**: A `.json.jbuilder` file that defines JSON structure using a Ruby DSL.
- **`json.extract!`**: Extracts specific attributes from an object.
- **Partial Rendering**: Reuse JSON fragments across endpoints with `json.partial!`.
- **Collection Rendering**: Render arrays of objects with `json.array!`.

## Real World Context
Jbuilder shines when your JSON structure doesn't mirror your database schema: nested objects, computed fields, conditional attributes, and complex relationships. It's included in Rails by default and follows the same view layer conventions as ERB templates.

## Deep Dive
### Basic Template

```ruby
# app/views/api/v1/products/show.json.jbuilder
json.extract! @product, :id, :name, :description
json.price @product.formatted_price
json.in_stock @product.stock_count > 0

json.category do
  json.extract! @product.category, :id, :name
end

json.links do
  json.self api_v1_product_url(@product)
  json.reviews api_v1_product_reviews_url(@product)
end
```

`json.extract!` pulls named attributes. Custom keys can use any Ruby expression. Nested objects use blocks.

### Collection Template

```ruby
# app/views/api/v1/products/index.json.jbuilder
json.data @products do |product|
  json.extract! product, :id, :name
  json.price product.formatted_price
  json.category product.category.name
end

json.meta do
  json.total @products.total_count
  json.page @products.current_page
  json.per_page @products.limit_value
end
```

Blocks iterate over collections. The envelope pattern (`data` + `meta`) is easy to implement.

### Partials

```ruby
# app/views/api/v1/products/_product.json.jbuilder
json.extract! product, :id, :name, :description
json.price product.formatted_price
json.category product.category&.name

# Reuse in index:
json.data @products, partial: "api/v1/products/product", as: :product

# Reuse in show:
json.partial! "api/v1/products/product", product: @product
```

Partials prevent duplication between index and show endpoints.

### Conditional Attributes

```ruby
json.extract! @product, :id, :name, :price
json.admin_notes @product.admin_notes if current_user&.admin?
json.edit_url edit_api_v1_product_url(@product) if current_user&.can_edit?(@product)
```

Conditional rendering lets you tailor responses based on the authenticated user's permissions.

## Common Pitfalls
1. **N+1 queries in templates** — Accessing associations in templates without eager loading triggers N+1 queries. Always use `includes` in the controller.
2. **Overly complex templates** — If a template exceeds 30-40 lines, extract partials or consider a serializer library.

## Best Practices
1. **Use partials for reusable fragments** — Define a `_product.json.jbuilder` and reuse it across endpoints.
2. **Eager load in the controller** — `@products = Product.includes(:category, :reviews).all`.

## Summary
- Jbuilder templates (`.json.jbuilder`) define JSON with a clean Ruby DSL.
- `json.extract!` pulls model attributes, blocks create nested objects.
- Partials share JSON fragments across endpoints.
- Conditional rendering tailors responses based on user permissions.
- Always eager load associations in controllers to prevent N+1 queries.

## Code Examples

**A Jbuilder template with nested objects, collections, computed values, and HATEOAS links**

```ruby
# app/views/api/v1/products/show.json.jbuilder
json.extract! @product, :id, :name, :description
json.price @product.formatted_price
json.in_stock @product.stock_count.positive?

json.category do
  json.extract! @product.category, :id, :name
end

json.reviews @product.reviews do |review|
  json.extract! review, :id, :body, :rating
  json.author review.author.name
end

json.links do
  json.self api_v1_product_url(@product)
end
```


## Resources

- [Jbuilder GitHub](https://github.com/rails/jbuilder) — Jbuilder gem documentation and examples

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*