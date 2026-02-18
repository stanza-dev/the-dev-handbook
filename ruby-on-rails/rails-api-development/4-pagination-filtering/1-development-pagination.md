---
source_course: "rails-api-development"
source_lesson: "rails-api-development-pagination"
---

# API Pagination Strategies

APIs must paginate large collections. Let's explore different strategies.

## Offset-Based Pagination

The most common approach using page numbers:

```ruby
# Using Kaminari gem
# Gemfile
gem 'kaminari'

# Controller
class Api::V1::ArticlesController < Api::V1::BaseController
  def index
    @articles = Article.published
                       .order(created_at: :desc)
                       .page(params[:page])
                       .per(params[:per_page] || 25)

    render json: {
      data: ArticleSerializer.new(@articles).as_json,
      meta: {
        current_page: @articles.current_page,
        total_pages: @articles.total_pages,
        total_count: @articles.total_count,
        per_page: @articles.limit_value
      },
      links: pagination_links(@articles)
    }
  end

  private

  def pagination_links(collection)
    {
      self: url_for(page: collection.current_page),
      first: url_for(page: 1),
      last: url_for(page: collection.total_pages),
      prev: collection.prev_page ? url_for(page: collection.prev_page) : nil,
      next: collection.next_page ? url_for(page: collection.next_page) : nil
    }
  end
end
```

Client usage:
```
GET /api/v1/articles?page=2&per_page=10
```

## Cursor-Based Pagination

Better for real-time data and large datasets:

```ruby
class Api::V1::ArticlesController < Api::V1::BaseController
  def index
    @articles = fetch_articles_with_cursor

    render json: {
      data: ArticleSerializer.new(@articles).as_json,
      meta: {
        has_more: @articles.size > limit,
        next_cursor: @articles.last&.id
      }
    }
  end

  private

  def fetch_articles_with_cursor
    articles = Article.published.order(id: :desc)

    if params[:cursor]
      articles = articles.where("id < ?", params[:cursor])
    end

    articles.limit(limit + 1).to_a.first(limit)
  end

  def limit
    [(params[:limit] || 25).to_i, 100].min
  end
end
```

Client usage:
```
GET /api/v1/articles?cursor=100&limit=25
```

## Link Headers (REST Standard)

```ruby
class Api::V1::ArticlesController < Api::V1::BaseController
  def index
    @articles = Article.page(params[:page]).per(25)

    set_pagination_headers(@articles)
    render json: @articles
  end

  private

  def set_pagination_headers(collection)
    links = []
    links << %(<#{url_for(page: 1)}>; rel="first")
    links << %(<#{url_for(page: collection.total_pages)}>; rel="last")
    links << %(<#{url_for(page: collection.prev_page)}>; rel="prev") if collection.prev_page
    links << %(<#{url_for(page: collection.next_page)}>; rel="next") if collection.next_page

    response.headers["Link"] = links.join(", ")
    response.headers["X-Total-Count"] = collection.total_count.to_s
    response.headers["X-Total-Pages"] = collection.total_pages.to_s
  end
end
```

## Comparison

| Approach | Pros | Cons |
|----------|------|------|
| Offset | Simple, familiar | Slow for large offsets, inconsistent with real-time data |
| Cursor | Fast, consistent | Can't jump to specific page |
| Link Headers | REST standard | Requires header parsing |

## Resources

- [Kaminari Gem](https://github.com/kaminari/kaminari) — Popular pagination library for Rails

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*