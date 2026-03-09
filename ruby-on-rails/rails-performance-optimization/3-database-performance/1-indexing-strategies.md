---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-indexing-strategies"
---

# Indexing Strategies

## Introduction
Database indexes are the single most impactful performance optimization you can make. A missing index can turn a 1ms query into a 10-second table scan. Understanding when and how to add indexes is essential for any Rails developer.

## Key Concepts
- **Index**: A data structure that speeds up lookups on specific columns, like a book's index.
- **Composite Index**: An index on multiple columns — column order determines which queries it accelerates.
- **Partial Index**: An index that only covers rows matching a condition, saving space and write overhead.

## Real World Context
Rails does not automatically add indexes on foreign key columns. Every `belongs_to` association generates queries on the foreign key, and without an index, those queries scan the entire table. This is one of the most common performance issues in new Rails apps.

## Deep Dive

### Creating Indexes

```ruby
class AddIndexes < ActiveRecord::Migration[8.1]
  def change
    # Foreign key index (Rails doesn't add this automatically!)
    add_index :posts, :user_id

    # Unique index
    add_index :users, :email, unique: true

    # Composite index (column order matters!)
    add_index :posts, [:user_id, :published_at]

    # Partial index (only indexes published posts)
    add_index :posts, :published_at, where: 'published = true'

    # Index with custom name
    add_index :orders, :customer_id, name: 'idx_orders_customer'
  end
end
```

### Composite Index Order

Column order in composite indexes matters:

```ruby
# Index on [:user_id, :created_at]

# Uses the index:
Post.where(user_id: 1)
Post.where(user_id: 1, created_at: Date.today)
Post.where(user_id: 1).order(created_at: :desc)

# Does NOT use the index efficiently:
Post.where(created_at: Date.today)  # Wrong column order!
```

Rule: put equality conditions first, then range/order columns.

### Index Costs

Indexes slow down writes:
- Every INSERT updates all indexes on that table
- Every UPDATE to indexed columns rebuilds affected indexes
- Indexes consume disk space

## Common Pitfalls
1. **Missing foreign key indexes** — Check every `belongs_to` association has a corresponding index on the foreign key column.
2. **Wrong composite index order** — An index on `[:user_id, :created_at]` cannot efficiently answer `WHERE created_at = ?` without `user_id`.

## Best Practices
1. **Index all foreign keys** — Run `bin/rails db:migrate:status` and verify every foreign key has an index.
2. **Use partial indexes for filtered queries** — If you frequently query `WHERE status = 'active'`, a partial index on that condition is smaller and faster.

## Summary
- Always index foreign key columns — Rails doesn't do this automatically.
- Composite index column order must match your query patterns.
- Partial indexes save space by only indexing matching rows.
- Indexes speed up reads but slow down writes — don't over-index.

## Code Examples

**Three index types — foreign key, composite for multi-column queries, and partial for filtered subsets**

```ruby
class AddPerformanceIndexes < ActiveRecord::Migration[8.1]
  def change
    # Foreign key (most common missing index)
    add_index :comments, :post_id

    # Composite for common query pattern
    add_index :orders, [:user_id, :created_at]

    # Partial index for filtered queries
    add_index :products, :price, where: 'active = true',
              name: 'idx_active_products_price'
  end
end
```


## Resources

- [Active Record Migrations — Indexes](https://guides.rubyonrails.org/active_record_migrations.html) — Official Rails guide on creating indexes in migrations

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*