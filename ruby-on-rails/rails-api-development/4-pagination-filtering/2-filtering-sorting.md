---
source_course: "rails-api-development"
source_lesson: "rails-api-development-filtering-sorting"
---

# Filtering and Sorting

Give clients the power to filter and sort API results.

## Basic Filtering

```ruby
class Api::V1::ArticlesController < Api::V1::BaseController
  def index
    @articles = Article.all
    @articles = apply_filters(@articles)
    @articles = apply_sorting(@articles)
    @articles = @articles.page(params[:page])

    render json: ArticleSerializer.new(@articles).as_json
  end

  private

  def apply_filters(scope)
    scope = scope.where(published: true) if params[:published] == "true"
    scope = scope.where(published: false) if params[:published] == "false"
    scope = scope.where(author_id: params[:author_id]) if params[:author_id]
    scope = scope.where(category_id: params[:category_id]) if params[:category_id]

    if params[:created_after]
      scope = scope.where("created_at >= ?", params[:created_after])
    end

    if params[:search]
      scope = scope.where("title ILIKE ?", "%#{params[:search]}%")
    end

    scope
  end

  def apply_sorting(scope)
    allowed_sorts = %w[created_at title views_count]
    sort_by = allowed_sorts.include?(params[:sort]) ? params[:sort] : "created_at"
    direction = params[:direction] == "asc" ? :asc : :desc

    scope.order(sort_by => direction)
  end
end
```

## Filter Object Pattern

```ruby
# app/filters/article_filter.rb
class ArticleFilter
  def initialize(params)
    @params = params
  end

  def apply(scope)
    scope
      .then { |s| filter_by_published(s) }
      .then { |s| filter_by_author(s) }
      .then { |s| filter_by_category(s) }
      .then { |s| filter_by_date_range(s) }
      .then { |s| filter_by_search(s) }
      .then { |s| filter_by_tags(s) }
  end

  private

  def filter_by_published(scope)
    return scope unless @params[:published].present?
    scope.where(published: @params[:published] == "true")
  end

  def filter_by_author(scope)
    return scope unless @params[:author_id].present?
    scope.where(author_id: @params[:author_id])
  end

  def filter_by_category(scope)
    return scope unless @params[:category_id].present?
    scope.where(category_id: @params[:category_id])
  end

  def filter_by_date_range(scope)
    scope = scope.where("created_at >= ?", @params[:from]) if @params[:from]
    scope = scope.where("created_at <= ?", @params[:to]) if @params[:to]
    scope
  end

  def filter_by_search(scope)
    return scope unless @params[:q].present?

    query = "%#{@params[:q]}%"
    scope.where("title ILIKE ? OR body ILIKE ?", query, query)
  end

  def filter_by_tags(scope)
    return scope unless @params[:tags].present?

    tag_ids = @params[:tags].split(",")
    scope.joins(:tags).where(tags: { id: tag_ids }).distinct
  end
end

# Usage
class Api::V1::ArticlesController < Api::V1::BaseController
  def index
    @articles = ArticleFilter.new(filter_params).apply(Article.all)
    @articles = ArticleSorter.new(sort_params).apply(@articles)
    @articles = @articles.page(params[:page])

    render json: ArticleSerializer.new(@articles).as_json
  end

  private

  def filter_params
    params.permit(:published, :author_id, :category_id, :from, :to, :q, :tags)
  end
end
```

## API Usage Examples

```
# Filter by author
GET /api/v1/articles?author_id=5

# Filter by multiple criteria
GET /api/v1/articles?published=true&category_id=3

# Search
GET /api/v1/articles?q=rails+tutorial

# Date range
GET /api/v1/articles?from=2024-01-01&to=2024-06-30

# Sort
GET /api/v1/articles?sort=views_count&direction=desc

# Combine everything
GET /api/v1/articles?published=true&sort=created_at&direction=desc&page=1&per_page=10
```

## Resources

- [Ransack Gem](https://github.com/activerecord-hackery/ransack) — Advanced search and filtering for Rails

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*