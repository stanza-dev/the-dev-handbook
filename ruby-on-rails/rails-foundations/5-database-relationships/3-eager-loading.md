---
source_course: "rails-foundations"
source_lesson: "rails-foundations-eager-loading"
---

# Eager Loading and N+1 Queries

The N+1 query problem is one of the most common performance issues in Rails applications. Let's understand it and learn how to fix it.

## The N+1 Problem

Consider displaying articles with their authors:

```ruby
# Controller
def index
  @articles = Article.all
end
```

```erb
<% @articles.each do |article| %>
  <h2><%= article.title %></h2>
  <p>By: <%= article.author.name %></p>  <!-- Triggers a query! -->
<% end %>
```

This generates:
```sql
-- 1 query for articles
SELECT * FROM articles;

-- N queries for authors (one per article!)
SELECT * FROM authors WHERE id = 1;
SELECT * FROM authors WHERE id = 2;
SELECT * FROM authors WHERE id = 3;
-- ... and so on
```

With 100 articles, that's **101 queries**!

## The Solution: Eager Loading

Load associated records in advance with `includes`:

```ruby
# Controller - Fixed!
def index
  @articles = Article.includes(:author)
end
```

Now only **2 queries**:
```sql
SELECT * FROM articles;
SELECT * FROM authors WHERE id IN (1, 2, 3, ...);
```

## Eager Loading Methods

### includes

The most common method - Rails decides the best strategy:

```ruby
# Single association
Article.includes(:author)

# Multiple associations
Article.includes(:author, :tags)

# Nested associations
Article.includes(comments: :author)

# Mix of both
Article.includes(:author, comments: [:author, :likes])
```

### preload

Forces separate queries (always uses IN clause):

```ruby
Article.preload(:comments)
# SELECT * FROM articles
# SELECT * FROM comments WHERE article_id IN (1, 2, 3)
```

### eager_load

Forces a single query with LEFT OUTER JOIN:

```ruby
Article.eager_load(:comments)
# SELECT articles.*, comments.*
# FROM articles
# LEFT OUTER JOIN comments ON comments.article_id = articles.id
```

Use `eager_load` when you need to filter by associated attributes:

```ruby
Article.eager_load(:comments).where(comments: { approved: true })
```

## Detecting N+1 Queries

### In Development

Use the **Bullet gem** to automatically detect N+1 queries:

```ruby
# Gemfile
group :development do
  gem 'bullet'
end
```

```ruby
# config/environments/development.rb
config.after_initialize do
  Bullet.enable = true
  Bullet.alert = true
  Bullet.console = true
end
```

### Watch the Logs

Look for repeated queries in your Rails log:

```
Article Load (0.5ms)  SELECT * FROM articles
Author Load (0.2ms)  SELECT * FROM authors WHERE id = 1
Author Load (0.2ms)  SELECT * FROM authors WHERE id = 2  # Repeated!
```

## Common Patterns

```ruby
# Index pages - preload what you'll display
def index
  @articles = Article.includes(:author, :tags).recent.limit(20)
end

# Show pages - load related data
def show
  @article = Article.includes(comments: :author).find(params[:id])
end
```

## Resources

- [Active Record Query Interface - Eager Loading](https://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations) — Guide to preventing N+1 queries

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*