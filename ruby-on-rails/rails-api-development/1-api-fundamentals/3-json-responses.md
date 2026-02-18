---
source_course: "rails-api-development"
source_lesson: "rails-api-development-json-responses"
---

# Rendering JSON Responses

Learn different approaches to rendering JSON in Rails APIs.

## Basic JSON Rendering

### render json: object

```ruby
def show
  @article = Article.find(params[:id])
  render json: @article
end
# Output: {"id":1,"title":"Hello","body":"...","created_at":"..."}
```

### With Options

```ruby
# Include only specific attributes
render json: @article, only: [:id, :title, :body]

# Exclude attributes
render json: @article, except: [:created_at, :updated_at]

# Include associations
render json: @article, include: [:author, :comments]

# Nested includes
render json: @article, include: { comments: { include: :author } }
```

## Custom JSON Structure

```ruby
def show
  @article = Article.find(params[:id])

  render json: {
    data: {
      id: @article.id,
      type: "article",
      attributes: {
        title: @article.title,
        body: @article.body,
        published: @article.published?,
        published_at: @article.published_at&.iso8601
      },
      relationships: {
        author: {
          id: @article.author_id,
          name: @article.author.name
        }
      }
    }
  }
end
```

## Using Jbuilder

Jbuilder provides a DSL for building JSON:

```ruby
# app/views/api/v1/articles/show.json.jbuilder
json.data do
  json.id @article.id
  json.type "article"

  json.attributes do
    json.title @article.title
    json.body @article.body
    json.published @article.published?
    json.created_at @article.created_at.iso8601
  end

  json.author do
    json.id @article.author.id
    json.name @article.author.name
  end

  json.comments @article.comments do |comment|
    json.id comment.id
    json.body comment.body
    json.author comment.author.name
  end
end
```

### Partial Templates

```ruby
# app/views/api/v1/articles/_article.json.jbuilder
json.id article.id
json.title article.title
json.body article.body
json.created_at article.created_at.iso8601

# app/views/api/v1/articles/index.json.jbuilder
json.data @articles, partial: "api/v1/articles/article", as: :article
json.meta do
  json.total @articles.total_count
  json.page @articles.current_page
end
```

## Using as_json

Override in models for consistent representation:

```ruby
class Article < ApplicationRecord
  def as_json(options = {})
    super(options.merge(
      only: [:id, :title, :body, :published],
      methods: [:reading_time, :excerpt],
      include: {
        author: { only: [:id, :name] }
      }
    ))
  end
end
```

## Error Responses

```ruby
class ApplicationController < ActionController::API
  rescue_from ActiveRecord::RecordNotFound, with: :not_found
  rescue_from ActiveRecord::RecordInvalid, with: :unprocessable_entity

  private

  def not_found(exception)
    render json: {
      error: "Not Found",
      message: exception.message
    }, status: :not_found
  end

  def unprocessable_entity(exception)
    render json: {
      error: "Validation Failed",
      messages: exception.record.errors.full_messages
    }, status: :unprocessable_entity
  end
end
```

## Resources

- [Jbuilder GitHub](https://github.com/rails/jbuilder) — Jbuilder gem for JSON generation

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*