---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-scopes-queries"
---

# Scopes and Query Objects

Organize complex queries into reusable, composable units for cleaner code.

## Scopes

Scopes are class methods that return `ActiveRecord::Relation`:

```ruby
class Article < ApplicationRecord
  # Simple scopes
  scope :published, -> { where(published: true) }
  scope :draft, -> { where(published: false) }
  scope :recent, -> { order(created_at: :desc) }
  scope :featured, -> { where(featured: true) }

  # Scopes with arguments
  scope :by_author, ->(author) { where(author: author) }
  scope :created_after, ->(date) { where("created_at > ?", date) }
  scope :with_tag, ->(tag) { joins(:tags).where(tags: { name: tag }) }

  # Scopes with optional arguments
  scope :by_status, ->(status = nil) {
    status ? where(status: status) : all
  }

  # Default scope (use sparingly)
  default_scope { order(created_at: :desc) }
end

# Usage - scopes are chainable
Article.published.recent.limit(10)
Article.published.featured.by_author(current_user)
Article.draft.created_after(1.week.ago)
```

## Class Methods vs Scopes

```ruby
class Article < ApplicationRecord
  # Scope (always returns relation)
  scope :published, -> { where(published: true) }

  # Class method (more flexible)
  def self.search(query)
    return all if query.blank?

    where("title ILIKE :q OR body ILIKE :q", q: "%#{query}%")
  end

  def self.popular
    where("views_count > ?", 100).order(views_count: :desc)
  end
end
```

## Query Objects

For complex queries, extract to dedicated objects:

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
      .then { |r| filter_by_date_range(r, params[:start_date], params[:end_date]) }
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

  def filter_by_date_range(relation, start_date, end_date)
    relation = relation.where("created_at >= ?", start_date) if start_date
    relation = relation.where("created_at <= ?", end_date) if end_date
    relation
  end

  def search_text(relation, query)
    return relation if query.blank?
    relation.where("title ILIKE :q OR body ILIKE :q", q: "%#{query}%")
  end

  def sort_by(relation, sort)
    case sort
    when "recent" then relation.order(created_at: :desc)
    when "popular" then relation.order(views_count: :desc)
    when "alphabetical" then relation.order(:title)
    else relation.order(created_at: :desc)
    end
  end
end

# Usage in controller
class ArticlesController < ApplicationController
  def index
    @articles = ArticleSearchQuery.new(Article.published)
                                  .call(search_params)
                                  .page(params[:page])
  end

  private

  def search_params
    params.permit(:status, :author_id, :start_date, :end_date, :query, :sort)
  end
end
```

## Avoiding Default Scopes

```ruby
# Default scopes can cause issues
class Article < ApplicationRecord
  default_scope { where(deleted: false) }  # Problematic!
end

# Better: Use a scope and apply explicitly
class Article < ApplicationRecord
  scope :active, -> { where(deleted: false) }
  scope :with_deleted, -> { unscope(where: :deleted) }
end
```

## Resources

- [Active Record Query Interface - Scopes](https://guides.rubyonrails.org/active_record_querying.html#scopes) — Official guide to scopes

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*