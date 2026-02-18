---
source_course: "rails-api-development"
source_lesson: "rails-api-development-versioning-strategies"
---

# API Versioning Strategies

As your API evolves, you need strategies to maintain backwards compatibility while adding new features.

## Why Version?

- Clients depend on specific response formats
- Breaking changes would break existing apps
- Different clients may need different versions
- Allows gradual migration to new versions

## Strategy 1: URL Path Versioning

Most common and explicit:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :articles
    end

    namespace :v2 do
      resources :articles
    end
  end
end
```

```
GET /api/v1/articles
GET /api/v2/articles
```

**Pros**: Clear, cacheable, easy to understand  
**Cons**: Duplicates routes, pollutes URLs

## Strategy 2: Header Versioning

```ruby
# config/routes.rb
namespace :api do
  scope module: :v2, constraints: ApiVersion.new(2) do
    resources :articles
  end

  scope module: :v1, constraints: ApiVersion.new(1, default: true) do
    resources :articles
  end
end

# lib/api_version.rb
class ApiVersion
  def initialize(version, default: false)
    @version = version
    @default = default
  end

  def matches?(request)
    @default || check_headers(request)
  end

  private

  def check_headers(request)
    accept = request.headers["Accept"]
    accept&.include?("version=#{@version}")
  end
end
```

Client sends:
```
GET /api/articles
Accept: application/vnd.myapi+json; version=2
```

**Pros**: Clean URLs  
**Cons**: Less visible, harder to test

## Strategy 3: Query Parameter

```ruby
namespace :api do
  scope module: :v2, constraints: ->(req) { req.params[:version] == "2" } do
    resources :articles
  end

  scope module: :v1 do
    resources :articles
  end
end
```

```
GET /api/articles?version=2
```

**Pros**: Easy to test, no URL changes  
**Cons**: Easy to forget, not RESTful

## Managing Version Code

### Inheritance Pattern

```ruby
# V1 - Original
module Api::V1
  class ArticlesController < BaseController
    def index
      render json: Article.all
    end
  end
end

# V2 - Inherits and overrides
module Api::V2
  class ArticlesController < Api::V1::ArticlesController
    # Override only what changed
    def index
      render json: ArticleSerializer.new(Article.all).as_json
    end

    # Inherited methods work as-is
  end
end
```

### Adapter Pattern

```ruby
# Core logic in one place
class ArticlesService
  def self.list(filters)
    Article.where(filters).order(:created_at)
  end
end

# Version-specific presentation
module Api::V1
  class ArticlesController < BaseController
    def index
      articles = ArticlesService.list(filter_params)
      render json: V1::ArticleSerializer.new(articles)
    end
  end
end

module Api::V2
  class ArticlesController < BaseController
    def index
      articles = ArticlesService.list(filter_params)
      render json: V2::ArticleSerializer.new(articles)
    end
  end
end
```

## Deprecation

```ruby
class Api::V1::BaseController < ApplicationController
  before_action :add_deprecation_warning

  private

  def add_deprecation_warning
    response.headers["Deprecation"] = "true"
    response.headers["Sunset"] = "Sat, 31 Dec 2024 23:59:59 GMT"
    response.headers["Link"] = '</api/v2>; rel="successor-version"'
  end
end
```

## Resources

- [API Versioning Best Practices](https://guides.rubyonrails.org/api_app.html) — Rails API documentation

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*