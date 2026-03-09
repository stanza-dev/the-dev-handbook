---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-query-optimization-techniques"
---

# Query Optimization Techniques

## Introduction
Beyond eager loading, Active Record offers many tools to reduce query count, memory usage, and response time. Selecting only needed columns, processing in batches, and using counter caches are essential techniques for any Rails developer.

## Key Concepts
- **select / pluck**: Fetch only the columns you need instead of loading full objects.
- **Batch Processing**: Process large record sets in chunks with `find_each` to avoid loading everything into memory.
- **Counter Cache**: Store association counts in a column to avoid COUNT queries.

## Real World Context
A reporting page that calls `User.all` to display just names and emails loads every column — including large text fields and serialized data. Using `select(:id, :name, :email)` can reduce memory usage by 90% and cut query time dramatically.

## Deep Dive

### Select Only Needed Columns

```ruby
# Bad: loads all columns
User.all

# Good: only what you need
User.select(:id, :name, :email)

# pluck returns plain arrays (even faster)
User.where(active: true).pluck(:email)
# Returns: ['alice@example.com', 'bob@example.com']
```

### Batch Processing

```ruby
# Bad: loads all records into memory
User.all.each { |user| user.send_newsletter }

# Good: processes in batches of 1000
User.find_each { |user| user.send_newsletter }

# Custom batch size
User.find_each(batch_size: 500) { |user| user.process }

# Work with batches of records
User.find_in_batches(batch_size: 1000) do |users|
  users.each { |u| u.update_stats }
end
```

### Counter Cache

```ruby
class Comment < ApplicationRecord
  belongs_to :post, counter_cache: true
end
```

Add the migration:

```ruby
class AddCommentsCountToPosts < ActiveRecord::Migration[8.1]
  def change
    add_column :posts, :comments_count, :integer, default: 0
  end
end
```

### exists? vs present?

```ruby
# Bad: loads all records just to check
if User.where(admin: true).present?

# Good: uses SQL EXISTS
if User.where(admin: true).exists?
```

## Common Pitfalls
1. **Using `.count` in loops** — Each call fires a COUNT query. Use counter caches or load counts with a single grouped query.
2. **Calling `.all.map` instead of `.pluck`** — `.pluck(:email)` returns an array of strings directly from the database. `.all.map(&:email)` instantiates every ActiveRecord object first.

## Best Practices
1. **Use `find_each` for background jobs** — Any job processing more than a few hundred records should use batch processing to keep memory stable.
2. **Add counter caches for frequently displayed counts** — If you show comment counts on a post listing, a counter cache eliminates N queries.

## Summary
- Use `select` or `pluck` to avoid loading unnecessary columns.
- Process large sets with `find_each` to keep memory constant.
- Counter caches store association counts to avoid repeated COUNT queries.
- Prefer `exists?` over `present?` for existence checks.

## Code Examples

**Three key optimization techniques — pluck for column selection, find_each for batching, exists? for existence checks**

```ruby
# Efficient: returns plain array from database
emails = User.where(active: true).pluck(:email)
# => ['alice@example.com', 'bob@example.com']

# Efficient: processes 1000 records at a time
User.find_each(batch_size: 1000) do |user|
  UserMailer.weekly_digest(user).deliver_later
end

# Efficient: single SQL EXISTS check
has_admins = User.where(role: :admin).exists?
```


## Resources

- [Active Record Query Interface — Calculations](https://guides.rubyonrails.org/active_record_querying.html#calculations) — Official Rails guide on pluck, count, exists? and other query optimizations

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*