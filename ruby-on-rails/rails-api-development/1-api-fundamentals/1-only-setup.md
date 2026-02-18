---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-only-setup"
---

# Creating API-Only Rails Applications

Rails can be configured as an API-only application, removing view-related middleware for a leaner, faster API server.

## Creating an API Application

```bash
rails new my_api --api
```

The `--api` flag:
- Configures ApplicationController to inherit from `ActionController::API`
- Removes browser-specific middleware (cookies, sessions, flash)
- Skips generating views, helpers, and assets
- Configures generators to skip view generation

## What Changes?

### Application Controller

```ruby
# Standard Rails
class ApplicationController < ActionController::Base
end

# API-only Rails
class ApplicationController < ActionController::API
end
```

`ActionController::API` includes:
- URL generation
- Redirects
- Rendering (JSON, etc.)
- Parameter filtering
- HTTP caching
- Basic authentication

It excludes:
- Cookie/session management
- Flash messages
- Asset helpers
- Form helpers

### Configuration

```ruby
# config/application.rb
module MyApi
  class Application < Rails::Application
    config.api_only = true
  end
end
```

## Converting Existing App to API

```ruby
# config/application.rb
config.api_only = true

# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
end
```

## Basic API Controller

```ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
    render json: @articles
  end

  def show
    @article = Article.find(params[:id])
    render json: @article
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      render json: @article, status: :created
    else
      render json: { errors: @article.errors }, status: :unprocessable_entity
    end
  end

  def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
      render json: @article
    else
      render json: { errors: @article.errors }, status: :unprocessable_entity
    end
  end

  def destroy
    @article = Article.find(params[:id])
    @article.destroy
    head :no_content
  end

  private

  def article_params
    params.require(:article).permit(:title, :body, :published)
  end
end
```

## HTTP Status Codes

Use appropriate status codes:

| Status | Symbol | Use When |
|--------|--------|----------|
| 200 | :ok | Successful GET/PUT |
| 201 | :created | Successful POST |
| 204 | :no_content | Successful DELETE |
| 400 | :bad_request | Malformed request |
| 401 | :unauthorized | Authentication required |
| 403 | :forbidden | Not allowed |
| 404 | :not_found | Resource doesn't exist |
| 422 | :unprocessable_entity | Validation failed |
| 500 | :internal_server_error | Server error |

## Resources

- [Using Rails for API-only Applications](https://guides.rubyonrails.org/api_app.html) — Official guide to Rails API applications

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*