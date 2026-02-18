---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-indexing-strategies"
---

# Indexing Strategies

Proper indexing is crucial for database performance.

## When to Add Indexes

Add indexes for:
- Foreign keys (Rails doesn't add these automatically!)
- Columns used in WHERE clauses
- Columns used in ORDER BY
- Columns used in GROUP BY
- Columns used in JOIN conditions

## Creating Indexes

```ruby
class AddIndexes < ActiveRecord::Migration[7.1]
  def change
    # Single column index
    add_index :posts, :user_id
    
    # Unique index
    add_index :users, :email, unique: true
    
    # Composite index (column order matters!)
    add_index :posts, [:user_id, :published_at]
    
    # Partial index (only index matching rows)
    add_index :posts, :published_at, where: 'published = true'
    
    # Index with custom name
    add_index :orders, :customer_id, name: 'idx_orders_customer'
  end
end
```

## Composite Index Order

Column order in composite indexes matters:

```ruby
# Index on [:user_id, :created_at]

# Uses the index:
Post.where(user_id: 1)
Post.where(user_id: 1, created_at: Date.today)
Post.where(user_id: 1).order(created_at: :desc)

# Does NOT use the index efficiently:
Post.where(created_at: Date.today)  # Wrong order!
```

Rule: Put equality conditions first, then range/order columns.

## Checking Index Usage

```ruby
# Use EXPLAIN to see query plan
Post.where(user_id: 1).explain

# In console:
ActiveRecord::Base.connection.execute(
  "EXPLAIN ANALYZE SELECT * FROM posts WHERE user_id = 1"
).to_a
```

## Finding Missing Indexes

```ruby
# Gem that suggests indexes
gem 'lol_dba'

# Run: rake db:find_indexes
```

## Index Costs

Indexes slow down writes, so don't over-index:
- Every INSERT updates all indexes
- Every UPDATE to indexed columns updates indexes
- Indexes use disk space

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*