---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-standards"
---

# API Response Standards

Consistent response formats make your API easier to use. Let's explore common standards.

## JSON:API Specification

A popular standard for API responses:

```json
{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "Rails API Guide",
      "body": "Content here...",
      "created_at": "2024-01-15T10:30:00Z"
    },
    "relationships": {
      "author": {
        "data": { "type": "users", "id": "5" }
      },
      "tags": {
        "data": [
          { "type": "tags", "id": "1" },
          { "type": "tags", "id": "2" }
        ]
      }
    }
  },
  "included": [
    {
      "type": "users",
      "id": "5",
      "attributes": { "name": "John Doe" }
    }
  ]
}
```

## Simple Envelope Pattern

A straightforward approach many APIs use:

```ruby
# Success responses
{
  "success": true,
  "data": { ... }
}

# Collection responses
{
  "success": true,
  "data": [ ... ],
  "meta": {
    "total": 100,
    "page": 1,
    "per_page": 25
  }
}

# Error responses
{
  "success": false,
  "error": {
    "code": "validation_error",
    "message": "Validation failed",
    "details": [
      { "field": "title", "message": "can't be blank" },
      { "field": "body", "message": "is too short" }
    ]
  }
}
```

## Implementing Consistent Responses

```ruby
# app/controllers/concerns/api_response.rb
module ApiResponse
  extend ActiveSupport::Concern

  def render_success(data, status: :ok, meta: nil)
    response = { success: true, data: data }
    response[:meta] = meta if meta
    render json: response, status: status
  end

  def render_error(message, status: :bad_request, code: nil, details: nil)
    error = { message: message }
    error[:code] = code if code
    error[:details] = details if details
    render json: { success: false, error: error }, status: status
  end

  def render_validation_errors(record)
    details = record.errors.map do |error|
      { field: error.attribute, message: error.message }
    end
    render_error("Validation failed", status: :unprocessable_entity,
                 code: "validation_error", details: details)
  end
end

# Usage in controller
class Api::V1::ArticlesController < Api::V1::BaseController
  include ApiResponse

  def index
    @articles = Article.published.page(params[:page])
    render_success(
      ArticleSerializer.new(@articles).as_json,
      meta: pagination_meta(@articles)
    )
  end

  def create
    @article = current_user.articles.build(article_params)

    if @article.save
      render_success(ArticleSerializer.new(@article).as_json, status: :created)
    else
      render_validation_errors(@article)
    end
  end

  private

  def pagination_meta(collection)
    {
      total: collection.total_count,
      page: collection.current_page,
      per_page: collection.limit_value,
      total_pages: collection.total_pages
    }
  end
end
```

## Error Codes

Use consistent error codes:

```ruby
module ErrorCodes
  VALIDATION_ERROR = "validation_error"
  NOT_FOUND = "not_found"
  UNAUTHORIZED = "unauthorized"
  FORBIDDEN = "forbidden"
  RATE_LIMITED = "rate_limited"
  SERVER_ERROR = "server_error"
end
```

## Resources

- [JSON:API Specification](https://jsonapi.org/) — The JSON:API specification

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*