---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-query-analysis"
---

# Query Analysis

Understanding how your database executes queries helps you optimize them.

## Using EXPLAIN

```ruby
# Basic explain
Post.where(user_id: 1).explain

# With analysis (actually runs the query)
Post.where(user_id: 1).explain(:analyze)
```

## Reading EXPLAIN Output (PostgreSQL)

```
Seq Scan on posts  (cost=0.00..1234.00 rows=100 width=200)
  Filter: (user_id = 1)
```

Key terms:
- **Seq Scan**: Full table scan (usually bad)
- **Index Scan**: Using an index (good)
- **cost**: Estimated cost (lower is better)
- **rows**: Estimated rows returned

## Good vs Bad Plans

```sql
-- BAD: Sequential scan
Seq Scan on posts (cost=0.00..25000.00 rows=500 width=100)
  Filter: (status = 'published')

-- GOOD: Index scan  
Index Scan using index_posts_on_status on posts (cost=0.00..50.00 rows=500 width=100)
  Index Cond: (status = 'published')
```

## Common Query Problems

### Functions Prevent Index Usage

```ruby
# BAD: Function on indexed column
User.where('LOWER(email) = ?', email.downcase)

# GOOD: Store normalized data or use expression index
User.where(email: email.downcase)

# Or create expression index
add_index :users, 'LOWER(email)'
```

### LIKE with Leading Wildcard

```ruby
# BAD: Can't use index
Post.where('title LIKE ?', '%rails%')

# GOOD: No leading wildcard
Post.where('title LIKE ?', 'rails%')

# For full-text search, use proper solution
Post.where('to_tsvector(title) @@ to_tsquery(?)', 'rails')
```

## Slow Query Log

```ruby
# config/environments/production.rb
config.active_record.warn_on_records_fetched_greater_than = 1000
```

```yaml
# config/database.yml (PostgreSQL)
production:
  variables:
    log_min_duration_statement: 1000  # Log queries > 1 second
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*