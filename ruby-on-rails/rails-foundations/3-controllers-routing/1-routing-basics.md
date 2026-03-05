---
source_course: "rails-foundations"
source_lesson: "rails-foundations-routing-basics"
---

# Rails Routing Fundamentals

## Introduction

The Rails router is the traffic controller of your application. It examines every incoming URL and directs the request to the correct controller action, turning web addresses into code execution.

## Key Concepts

- **Router**: The component that maps incoming URLs to controller actions, defined in `config/routes.rb`.
- **RESTful resources**: `resources :articles` generates 7 standard CRUD routes in one line.
- **Path helpers**: Auto-generated methods like `articles_path` and `article_path(@article)` that produce URL strings.
- **Route parameters**: Dynamic URL segments like `:id` in `/articles/:id` that become available in `params`.

## Real World Context

Clean, predictable URLs are essential for SEO and user experience. Rails' RESTful routing conventions mean every Rails developer knows that `GET /articles/1` shows an article and `DELETE /articles/1` deletes it, without reading any documentation.

## Deep Dive

### The Routes File

```ruby
Rails.application.routes.draw do
  root "pages#home"

  resources :articles

  get "/about", to: "pages#about"
end
```

### RESTful Resource Routes

`resources :articles` generates:

| HTTP Verb | Path | Controller#Action | Purpose |
|-----------|------|-------------------|----------|
| GET | /articles | articles#index | List all |
| GET | /articles/new | articles#new | New form |
| POST | /articles | articles#create | Create |
| GET | /articles/:id | articles#show | Show one |
| GET | /articles/:id/edit | articles#edit | Edit form |
| PATCH/PUT | /articles/:id | articles#update | Update |
| DELETE | /articles/:id | articles#destroy | Delete |

### Path and URL Helpers

```ruby
articles_path          # => "/articles"
new_article_path       # => "/articles/new"
article_path(1)        # => "/articles/1"
edit_article_path(1)   # => "/articles/1/edit"
article_url(@article)  # => "http://localhost:3000/articles/1"
```

### Viewing Routes

```bash
bin/rails routes
bin/rails routes -g articles  # Filter by pattern
```

## Common Pitfalls

- **Route order matters**: Routes are matched top-to-bottom. A catch-all route defined too early can swallow more specific routes.
- **Forgetting `_path` vs `_url`**: Use `_path` for links within your app and `_url` for emails and redirects that need full URLs.
- **Over-nesting resources**: Deeply nested routes like `/users/1/posts/2/comments/3` are hard to manage. Keep nesting to one level.

## Best Practices

- Use `resources` for standard CRUD operations instead of defining routes manually.
- Run `bin/rails routes` frequently to verify your routes are correct.
- Use `_path` helpers instead of hardcoded URL strings.

## Summary

- All routes are defined in `config/routes.rb`.
- `resources :articles` generates 7 RESTful routes for CRUD operations.
- Rails generates path helpers like `articles_path` and `article_path(@article)` automatically.
- Use `bin/rails routes` to inspect all defined routes.
- Route parameters (`:id`) are accessible via the `params` hash in controllers.

## Code Examples

**The routes file maps URLs to controller actions. resources :articles generates all 7 RESTful routes and corresponding path helpers.**

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root "pages#home"
  resources :articles
  get "/about", to: "pages#about"
end

# Generated helpers:
# articles_path     => "/articles"
# article_path(1)   => "/articles/1"
# new_article_path  => "/articles/new"
```


## Resources

- [Rails Routing from the Outside In](https://guides.rubyonrails.org/routing.html) — Complete guide to Rails routing

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*