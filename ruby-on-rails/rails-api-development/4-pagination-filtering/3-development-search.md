---
source_course: "rails-api-development"
source_lesson: "rails-api-development-search"
---

# Search and Sorting

## Introduction
Beyond simple filtering, APIs need full-text search and flexible sorting. Rails provides basic search capabilities through SQL, and gems like pg_search add more powerful full-text search with PostgreSQL.

## Key Concepts
- **ILIKE**: PostgreSQL case-insensitive pattern matching for simple search.
- **Full-Text Search**: PostgreSQL's built-in text search with ranking and stemming.
- **Sort Parameters**: Letting clients control result ordering via query parameters.
- **pg_search**: A gem that wraps PostgreSQL full-text search in a clean Ruby API.

## Real World Context
Users expect to search APIs by keyword and sort results by relevance, date, price, or popularity. Simple ILIKE works for small datasets, but full-text search is needed when you have thousands of records with multi-word queries.

## Deep Dive
### Simple Search with ILIKE

```ruby
scope :search, ->(query) {
  where("name ILIKE :q OR description ILIKE :q",
        q: "%#{sanitize_sql_like(query)}%")
}
```

ILIKE is case-insensitive and works for simple substring matching. It's sufficient for small datasets but doesn't support ranking or stemming.

### Full-Text Search with pg_search

```ruby
# Gemfile
gem "pg_search"

# app/models/product.rb
class Product < ApplicationRecord
  include PgSearch::Model

  pg_search_scope :full_search,
    against: { name: "A", description: "B" },
    using: { tsearch: { prefix: true, dictionary: "english" } }
end
```

The `against` hash weights fields (A = highest). The `tsearch` option enables stemming ("running" matches "run") and prefix matching ("prod" matches "product").

### Flexible Sorting

```ruby
ALLOWED_SORTS = {
  "newest" => { created_at: :desc },
  "oldest" => { created_at: :asc },
  "price_low" => { price: :asc },
  "price_high" => { price: :desc },
  "name" => { name: :asc }
}.freeze

def apply_sort(scope)
  sort_config = ALLOWED_SORTS[params[:sort]] || { id: :asc }
  scope.order(sort_config)
end
```

Whitelisting sort options prevents arbitrary column sorting which could expose data or cause performance issues.

## Common Pitfalls
1. **Not indexing search columns** — ILIKE on unindexed columns is slow. Add a GIN trigram index for ILIKE, or use pg_search's built-in tsvector indexing.
2. **Allowing arbitrary sort columns** — Only whitelist known sort options to prevent information leakage.

## Best Practices
1. **Start with ILIKE, upgrade to pg_search** — ILIKE is fine for small datasets. Switch to pg_search when you need ranking, stemming, or better performance.
2. **Add database indexes for search** — `add_index :products, :name, using: :gin, opclass: :gin_trgm_ops` for ILIKE performance.

## Summary
- ILIKE provides simple case-insensitive search for small datasets.
- pg_search wraps PostgreSQL full-text search with ranking and stemming.
- Whitelist sort options to prevent arbitrary column sorting.
- Add GIN indexes for search performance.

## Code Examples

**PostgreSQL full-text search with field weighting and English stemming via pg_search**

```ruby
class Product < ApplicationRecord
  include PgSearch::Model

  pg_search_scope :full_search,
    against: { name: "A", description: "B" },
    using: {
      tsearch: { prefix: true, dictionary: "english" }
    }
end

# Product.full_search("wireless headphone")
# Matches: "Wireless Noise-Cancelling Headphones"
# Ranked by relevance (name matches weighted higher)
```


## Resources

- [pg_search Gem](https://github.com/Casecommons/pg_search) — pg_search gem for PostgreSQL full-text search in Rails

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*