---
source_course: "rails-foundations"
source_lesson: "rails-foundations-routing-basics"
---

# Rails Routing Fundamentals

The Rails router is the traffic controller of your application. It examines incoming URLs and directs them to the appropriate controller action.

## How Routing Works

When a user visits `/articles/1`, Rails:
1. Receives the HTTP request (GET /articles/1)
2. Consults `config/routes.rb`
3. Matches the URL to a route
4. Calls the corresponding controller action
5. Passes any URL parameters (like `id: 1`)

## The Routes File

All routes are defined in `config/routes.rb`:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  # Your routes go here
end
```

## Basic Route Types

### Root Route

The homepage of your application:

```ruby
root "pages#home"
# GET / => PagesController#home
```

### Simple Routes

Map a URL to a controller action:

```ruby
get "/about", to: "pages#about"
# GET /about => PagesController#about

post "/contact", to: "messages#create"
# POST /contact => MessagesController#create
```

### RESTful Resource Routes

The most common pattern in Rails - one line creates 7 routes:

```ruby
resources :articles
```

This generates:

| HTTP Verb | Path | Controller#Action | Purpose |
|-----------|------|-------------------|----------|
| GET | /articles | articles#index | List all |
| GET | /articles/new | articles#new | New form |
| POST | /articles | articles#create | Create |
| GET | /articles/:id | articles#show | Show one |
| GET | /articles/:id/edit | articles#edit | Edit form |
| PATCH/PUT | /articles/:id | articles#update | Update |
| DELETE | /articles/:id | articles#destroy | Delete |

## Path and URL Helpers

Rails generates helper methods for your routes:

```ruby
# For resources :articles
articles_path        # => "/articles"
articles_url         # => "http://localhost:3000/articles"

new_article_path     # => "/articles/new"

article_path(1)      # => "/articles/1"
article_path(@article)  # => "/articles/1" (uses article.id)

edit_article_path(1) # => "/articles/1/edit"
```

Use `_path` for relative URLs and `_url` for absolute URLs.

## Viewing Your Routes

```bash
# List all routes
bin/rails routes

# Filter routes
bin/rails routes -g articles  # Routes matching 'articles'

# In browser (development only)
# Visit /rails/info/routes
```

## Route Parameters

Capture dynamic segments from URLs:

```ruby
get "/users/:id", to: "users#show"
# /users/42 => params[:id] = "42"

get "/posts/:year/:month", to: "posts#archive"
# /posts/2024/03 => params[:year] = "2024", params[:month] = "03"
```

## Resources

- [Rails Routing from the Outside In](https://guides.rubyonrails.org/routing.html) — Complete guide to Rails routing

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*