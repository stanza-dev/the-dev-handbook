---
source_course: "rails-foundations"
source_lesson: "rails-foundations-crud-operations"
---

# CRUD Operations with Active Record

CRUD stands for **Create, Read, Update, Delete** - the four basic database operations. Active Record makes these operations intuitive and Ruby-like.

## Create

There are several ways to create records:

### Using `new` and `save`

```ruby
# Create an instance (not saved yet)
article = Article.new
article.title = "My Title"
article.body = "Article content here"
article.save  # Saves to database
# => true (success) or false (validation failed)
```

### Using `new` with a hash

```ruby
article = Article.new(title: "My Title", body: "Content")
article.save
```

### Using `create` (new + save in one step)

```ruby
article = Article.create(title: "My Title", body: "Content")
# Returns the article (saved or not)
```

### Using `create!` (raises exception on failure)

```ruby
article = Article.create!(title: "My Title", body: "Content")
# Raises ActiveRecord::RecordInvalid if validation fails
```

## Read

Active Record provides many methods to query data:

### Finding by ID

```ruby
# Find a single record by ID
article = Article.find(1)
# => #<Article id: 1, title: "My Title"...>

# Find raises RecordNotFound if not found
Article.find(999)  # => ActiveRecord::RecordNotFound

# Use find_by to return nil instead
article = Article.find_by(id: 999)
# => nil
```

### Finding by attributes

```ruby
# Find first matching record
article = Article.find_by(title: "My Title")

# Find with multiple conditions
article = Article.find_by(title: "My Title", published: true)
```

### Retrieving multiple records

```ruby
# All records
articles = Article.all

# First and last
first_article = Article.first
last_article = Article.last

# Specific number
recent = Article.last(5)  # Last 5 articles
```

### Filtering with `where`

```ruby
# Find all published articles
published = Article.where(published: true)

# Multiple conditions
Article.where(published: true, author: "Alice")

# Using SQL conditions
Article.where("created_at > ?", 1.week.ago)

# Chaining queries
Article.where(published: true).where("views > ?", 100)
```

## Update

### Updating a single record

```ruby
article = Article.find(1)

# Update individual attributes
article.title = "New Title"
article.save

# Update with a hash
article.update(title: "New Title", body: "New content")

# update! raises exception on failure
article.update!(title: "New Title")
```

### Updating multiple records

```ruby
# Update all matching records
Article.where(published: false).update_all(published: true)
```

## Delete

### Deleting a single record

```ruby
article = Article.find(1)

# destroy runs callbacks and validations
article.destroy

# delete skips callbacks (faster but dangerous)
article.delete
```

### Deleting multiple records

```ruby
# Destroy all matching (runs callbacks)
Article.where(published: false).destroy_all

# Delete all matching (skips callbacks)
Article.where(published: false).delete_all
```

## Checking Persistence State

```ruby
article = Article.new(title: "Draft")
article.new_record?   # => true
article.persisted?    # => false

article.save
article.new_record?   # => false
article.persisted?    # => true
```

## Resources

- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) — Complete guide to querying with Active Record

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*