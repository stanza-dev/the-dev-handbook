---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-query-optimization-techniques"
---

# Query Optimization Techniques

Beyond eager loading, there are many ways to optimize queries.

## Select Only Needed Columns

```ruby
# Bad: loads all columns
User.all

# Good: only what you need
User.select(:id, :name, :email)

# With pluck for simple values
User.where(active: true).pluck(:email)
# Returns: ['a@example.com', 'b@example.com']
```

## Use Proper Indexes

```ruby
# Add indexes for columns you query frequently
class AddIndexesToPosts < ActiveRecord::Migration[7.1]
  def change
    add_index :posts, :user_id
    add_index :posts, :published_at
    add_index :posts, [:user_id, :published_at]  # Composite
    add_index :posts, :title, unique: true
  end
end
```

## Batch Processing

```ruby
# Bad: loads all records into memory
User.all.each { |user| user.send_newsletter }

# Good: processes in batches of 1000
User.find_each do |user|
  user.send_newsletter
end

# Custom batch size
User.find_each(batch_size: 500) do |user|
  user.process
end

# Work with batches of records
User.find_in_batches(batch_size: 1000) do |users|
  users.each { |u| u.update_stats }
end
```

## Avoid N+1 in Counters

```ruby
# Bad: N+1 for counts
@posts.each { |p| p.comments.count }

# Good: use counter cache
class Comment < ApplicationRecord
  belongs_to :post, counter_cache: true
end

# Add migration for counter column
add_column :posts, :comments_count, :integer, default: 0

# Now just access the attribute
@posts.each { |p| p.comments_count }
```

## exists? vs present?

```ruby
# Bad: loads all records just to check
if User.where(admin: true).present?

# Good: uses EXISTS query
if User.where(admin: true).exists?
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*