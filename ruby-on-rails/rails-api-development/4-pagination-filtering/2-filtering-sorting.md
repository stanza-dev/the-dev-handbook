---
source_course: "rails-api-development"
source_lesson: "rails-api-development-filtering-sorting"
---

# Query Filtering and Scoping

## Introduction
API clients need to filter collections by various criteria: category, price range, status, date range, and more. Rails scopes and query objects provide clean, composable patterns for building filtered queries from request parameters.

## Key Concepts
- **Scope**: A named query fragment on a model that can be chained.
- **Query Object**: A dedicated class that builds complex queries from parameters.
- **Composable Queries**: Building queries by chaining multiple conditions.
- **Parameter Whitelisting**: Only allowing known filter parameters to prevent SQL injection.

## Real World Context
Real APIs need flexible filtering: "Show me products in the Electronics category, priced under $50, that are in stock." Scopes make this composable: each filter is independent and can be applied in any combination.

## Deep Dive
### Model Scopes

```ruby
class Product < ApplicationRecord
  scope :by_category, ->(category_id) { where(category_id: category_id) }
  scope :price_range, ->(min, max) { where(price: min..max) }
  scope :in_stock, -> { where("stock_count > 0") }
  scope :search, ->(query) { where("name ILIKE ?", "%#{sanitize_sql_like(query)}%") }
  scope :recent, -> { order(created_at: :desc) }
end
```

Each scope is a chainable query fragment. They compose cleanly:

```ruby
Product.by_category(3).price_range(1000, 5000).in_stock.recent
```

### Controller Filtering

```ruby
def index
  products = Product.all
  products = products.by_category(params[:category_id]) if params[:category_id]
  products = products.price_range(params[:min_price], params[:max_price]) if params[:min_price] && params[:max_price]
  products = products.in_stock if params[:in_stock] == "true"
  products = products.search(params[:q]) if params[:q].present?

  pagy, products = pagy(products)
  render json: { data: products, meta: pagy_metadata(pagy) }
end
```

Each parameter conditionally applies a scope. The query is only as complex as the filters requested.

### Query Object Pattern

For complex filtering, extract to a query object:

```ruby
class ProductQuery
  ALLOWED_FILTERS = %w[category_id min_price max_price in_stock q sort].freeze

  def initialize(params)
    @params = params.slice(*ALLOWED_FILTERS)
  end

  def call
    scope = Product.all
    scope = scope.by_category(@params[:category_id]) if @params[:category_id]
    scope = scope.price_range(@params[:min_price], @params[:max_price]) if @params[:min_price]
    scope = scope.in_stock if @params[:in_stock] == "true"
    scope = scope.search(@params[:q]) if @params[:q].present?
    scope = apply_sort(scope)
    scope
  end

  private

  def apply_sort(scope)
    case @params[:sort]
    when "price_asc" then scope.order(price: :asc)
    when "price_desc" then scope.order(price: :desc)
    when "newest" then scope.order(created_at: :desc)
    else scope.order(:id)
    end
  end
end
```

Usage: `products = ProductQuery.new(params).call`.

## Common Pitfalls
1. **SQL injection through filters** — Always use parameterized queries. Never interpolate params into SQL strings.
2. **Not whitelisting filter params** — Accept only known parameters to prevent unexpected query behavior.

## Best Practices
1. **Use `sanitize_sql_like` for LIKE queries** — Prevents special characters (%, _) in user input from acting as wildcards.
2. **Extract query objects for complex filtering** — When a controller has more than 3-4 filter conditions, move to a query object.

## Summary
- Model scopes define reusable, chainable query fragments.
- Controllers apply scopes conditionally based on request parameters.
- Query objects centralize complex filtering logic with parameter whitelisting.
- Always sanitize LIKE queries and whitelist filter parameters.

## Code Examples

**Composable scopes that can be chained in any combination for flexible API filtering**

```ruby
# Composable model scopes for API filtering
class Product < ApplicationRecord
  scope :by_category, ->(id) { where(category_id: id) }
  scope :price_range, ->(min, max) { where(price: min..max) }
  scope :in_stock, -> { where("stock_count > 0") }
  scope :search, ->(q) {
    where("name ILIKE ?", "%#{sanitize_sql_like(q)}%")
  }
end

# Chain them: Product.by_category(3).in_stock.search("phone")
```


## Resources

- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) — Rails guide on scopes, where clauses, and query methods

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*