---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-routing"
---

# API Routing and Namespacing

Organize your API with namespaces and proper routing conventions.

## Basic API Routing

```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :articles, only: [:index, :show, :create, :update, :destroy]
  resources :users, only: [:show, :create]
end
```

## Namespaced Routes

Group API routes under a namespace:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :api do
    resources :articles
    resources :users
  end
end
```

This creates:
- Routes prefixed with `/api/`
- Controllers in `app/controllers/api/`
- Expected controller names like `Api::ArticlesController`

```ruby
# app/controllers/api/articles_controller.rb
module Api
  class ArticlesController < ApplicationController
    def index
      @articles = Article.all
      render json: @articles
    end
  end
end
```

## Versioned API Routes

```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :articles
      resources :users
    end

    namespace :v2 do
      resources :articles
      resources :users
    end
  end
end
```

This creates:
- `/api/v1/articles` → `Api::V1::ArticlesController`
- `/api/v2/articles` → `Api::V2::ArticlesController`

### Directory Structure

```
app/controllers/
├── api/
│   ├── v1/
│   │   ├── articles_controller.rb
│   │   ├── users_controller.rb
│   │   └── base_controller.rb
│   └── v2/
│       ├── articles_controller.rb
│       └── base_controller.rb
└── application_controller.rb
```

### Base Controller Per Version

```ruby
# app/controllers/api/v1/base_controller.rb
module Api
  module V1
    class BaseController < ApplicationController
      # V1-specific behavior
    end
  end
end

# app/controllers/api/v1/articles_controller.rb
module Api
  module V1
    class ArticlesController < BaseController
      def index
        @articles = Article.all
        render json: @articles
      end
    end
  end
end
```

## Scope Without Module

URL prefix without controller namespace:

```ruby
scope '/api/v1' do
  resources :articles  # Uses ArticlesController (no Api::V1 module)
end
```

## Nested Resources

```ruby
namespace :api do
  namespace :v1 do
    resources :articles do
      resources :comments, only: [:index, :create, :destroy]
    end

    resources :users do
      resources :articles, only: [:index]
    end
  end
end
```

Creates:
- GET `/api/v1/articles/:article_id/comments`
- POST `/api/v1/articles/:article_id/comments`
- GET `/api/v1/users/:user_id/articles`

## Custom Routes

```ruby
namespace :api do
  namespace :v1 do
    resources :articles do
      member do
        post :publish
        delete :archive
      end

      collection do
        get :search
        get :featured
      end
    end
  end
end
```

Creates:
- POST `/api/v1/articles/:id/publish`
- DELETE `/api/v1/articles/:id/archive`
- GET `/api/v1/articles/search`
- GET `/api/v1/articles/featured`

## Resources

- [Rails Routing from the Outside In](https://guides.rubyonrails.org/routing.html) — Complete guide to Rails routing

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*