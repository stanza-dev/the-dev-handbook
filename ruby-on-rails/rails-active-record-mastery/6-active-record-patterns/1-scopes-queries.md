---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-scopes-queries"
---

# Scopes and Query Objects

## Introduction
As applications grow, query logic scattered across controllers and models becomes hard to maintain. Scopes and query objects organize complex queries into reusable, composable units. Scopes are lightweight and built into Active Record; query objects are a design pattern for when scopes are not enough.

## Key Concepts
- **Scope**: A class-level method defined with `scope :name, -> { ... }` that returns an `ActiveRecord::Relation` and is chainable.
- **Query object**: A plain Ruby class that encapsulates complex query logic with multiple parameters.
- **`default_scope`**: An automatically-applied scope (use with extreme caution).
- **Composability**: The ability to chain scopes together because each returns a Relation.

## Real World Context
Every non-trivial Rails application has search pages, filtered lists, and dashboards. Without scopes, you end up with fat controllers full of `where` chains. Without query objects, you end up with scopes that accept too many arguments. Clean query architecture is a hallmark of well-maintained Rails codebases.

## Deep Dive

### Scopes

Scopes are class methods that return `ActiveRecord::Relation`:

```ruby
class Article < ApplicationRecord
  scope :published, -> { where(published: true) }
  scope :draft, -> { where(published: false) }
  scope :recent, -> { order(created_at: :desc) }
  scope :featured, -> { where(featured: true) }

  # Scopes with arguments
  scope :by_author, ->(author) { where(author: author) }
  scope :created_after, ->(date) { where("created_at > ?", date) }

  # Default scope (use sparingly!)
  default_scope { order(created_at: :desc) }
end

# Chainable usage
Article.published.recent.limit(10)
Article.published.featured.by_author(current_user)
```

Because every scope returns a Relation, you can chain them freely. This composability is the key advantage.

### Class Methods vs Scopes

```ruby
class Article < ApplicationRecord
  # Scope (always returns relation, even for nil)
  scope :published, -> { where(published: true) }

  # Class method (more flexible for complex logic)
  def self.search(query)
    return all if query.blank?
    where("title ILIKE :q OR body ILIKE :q", q: "%#{query}%")
  end
end
```

Use scopes for simple conditions and class methods for anything that needs branching logic or early returns.

### Query Objects

For complex queries with many parameters, extract to a dedicated class:

```ruby
# app/queries/article_search_query.rb
class ArticleSearchQuery
  def initialize(relation = Article.all)
    @relation = relation
  end

  def call(params)
    @relation
      .then { |r| filter_by_status(r, params[:status]) }
      .then { |r| filter_by_author(r, params[:author_id]) }
      .then { |r| search_text(r, params[:query]) }
      .then { |r| sort_by(r, params[:sort]) }
  end

  private

  def filter_by_status(relation, status)
    return relation if status.blank?
    relation.where(status: status)
  end

  def filter_by_author(relation, author_id)
    return relation if author_id.blank?
    relation.where(author_id: author_id)
  end

  def search_text(relation, query)
    return relation if query.blank?
    relation.where("title ILIKE :q OR body ILIKE :q", q: "%#{query}%")
  end

  def sort_by(relation, sort)
    case sort
    when "recent" then relation.order(created_at: :desc)
    when "popular" then relation.order(views_count: :desc)
    else relation.order(created_at: :desc)
    end
  end
end

# Usage in controller
@articles = ArticleSearchQuery.new(Article.published).call(search_params)
```

### Avoiding Default Scopes

```ruby
# Problematic default scope
class Article < ApplicationRecord
  default_scope { where(deleted: false) }  # Dangerous!
end

# Better: explicit scope
class Article < ApplicationRecord
  scope :active, -> { where(deleted: false) }
  scope :with_deleted, -> { unscope(where: :deleted) }
end
```

Default scopes affect every query, including association loading and `unscoped` is easy to forget.

## Common Pitfalls
1. **Default scopes causing unexpected behavior** -- They affect all queries, including admin views and background jobs. Prefer explicit scopes.
2. **Scopes that return nil** -- A scope should always return a Relation. If a scope might return nil (e.g., from an `if` without `else`), use a class method instead.
3. **Too many parameters in scopes** -- If a scope needs more than 2 arguments, it is time for a query object.

## Best Practices
1. **Use scopes for simple, frequently-used conditions** -- `published`, `recent`, `active` are good scope candidates.
2. **Use query objects for search/filter pages** -- Any endpoint with 3+ filter parameters benefits from a query object.
3. **Avoid default_scope** -- It causes more problems than it solves. Use explicit scopes instead.

## Summary
- Scopes return `ActiveRecord::Relation` and are chainable.
- Use class methods for complex logic that needs branching.
- Query objects encapsulate multi-parameter search and filtering logic.
- Avoid `default_scope` -- use explicit scopes instead.
- Scopes and query objects keep controllers thin and queries reusable.

## Code Examples

**Scopes are chainable class methods that return ActiveRecord::Relation -- combine them freely for composable queries**

```ruby
class Article < ApplicationRecord
  scope :published, -> { where(published: true) }
  scope :recent, -> { order(created_at: :desc) }
  scope :by_author, ->(author) { where(author: author) }

  def self.search(query)
    return all if query.blank?
    where("title ILIKE :q", q: "%#{query}%")
  end
end

# Chainable usage
Article.published.recent.by_author(user).limit(10)
```


## Resources

- [Active Record Query Interface - Scopes](https://guides.rubyonrails.org/active_record_querying.html#scopes) — Official guide to scopes

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*