---
source_course: "rails-api-development"
source_lesson: "rails-api-development-pagination"
---

# API Pagination

## Introduction
APIs serving collections must paginate responses to avoid returning thousands of records in a single request. Rails has excellent pagination gems that integrate cleanly with API responses.

## Key Concepts
- **Offset Pagination**: Uses `page` and `per_page` parameters. Simple but slow for deep pages.
- **Cursor Pagination**: Uses an opaque cursor pointing to the last item. Fast and consistent.
- **Pagy**: A fast, lightweight pagination gem with API-friendly features.
- **Link Headers**: Pagination metadata returned in HTTP headers following the RFC 5988 standard.

## Real World Context
Every API endpoint that returns collections needs pagination. Without it, endpoints become slower as data grows, clients download unnecessary data, and servers waste memory. GitHub, Stripe, and Shopify all use pagination in their APIs.

## Deep Dive
### Offset Pagination with Pagy

```ruby
# Gemfile
gem "pagy"
```

```ruby
# app/controllers/api/v1/products_controller.rb
class Api::V1::ProductsController < ApplicationController
  include Pagy::Backend

  def index
    pagy, products = pagy(Product.all, items: params[:per_page] || 25)

    render json: {
      data: products,
      meta: {
        current_page: pagy.page,
        total_pages: pagy.pages,
        total_count: pagy.count,
        per_page: pagy.items
      }
    }
  end
end
```

Pagy is significantly faster than Kaminari (15x) and WillPaginate (40x) in benchmarks. The meta block gives clients all the information they need for pagination controls.

### Cursor Pagination

For large datasets, cursor pagination is more efficient:

```ruby
def index
  limit = (params[:limit] || 25).to_i.clamp(1, 100)
  products = Product.order(:id)

  if params[:after]
    products = products.where("id > ?", params[:after])
  end

  products = products.limit(limit + 1) # Fetch one extra to check for next page
  has_next = products.size > limit
  products = products.first(limit)

  render json: {
    data: products,
    meta: {
      has_next_page: has_next,
      next_cursor: has_next ? products.last.id : nil
    }
  }
end
```

Cursor pagination uses the last item's ID as a cursor. It doesn't suffer from the offset problem where deep pages require scanning many rows.

## Common Pitfalls
1. **Not clamping per_page** — Without limits, clients can request `per_page=100000` and crash your server.
2. **Using offset for large tables** — `OFFSET 10000` requires the database to scan 10,000 rows before returning results.

## Best Practices
1. **Default to 25 items per page** — A sensible default that balances payload size and request count.
2. **Always clamp the limit** — `(params[:limit] || 25).to_i.clamp(1, 100)` prevents abuse.

## Summary
- Use Pagy for fast offset pagination with API-friendly metadata.
- Use cursor pagination for large datasets where offset becomes slow.
- Always clamp page size to prevent abuse.
- Include pagination metadata in the response for client navigation.

## Code Examples

**Cursor pagination fetching one extra record to determine if there's a next page**

```ruby
# Cursor pagination — efficient for large datasets
def index
  limit = (params[:limit] || 25).to_i.clamp(1, 100)
  scope = Product.order(:id)
  scope = scope.where("id > ?", params[:after]) if params[:after]
  items = scope.limit(limit + 1).to_a
  has_next = items.size > limit
  items = items.first(limit)

  render json: {
    data: items,
    meta: { has_next_page: has_next, next_cursor: items.last&.id }
  }
end
```


## Resources

- [Pagy Gem](https://github.com/ddnexus/pagy) — Pagy documentation — the fastest pagination gem for Rails

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*