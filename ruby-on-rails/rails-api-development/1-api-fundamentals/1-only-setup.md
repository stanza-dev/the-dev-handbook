---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-only-setup"
---

# Creating API-Only Rails Applications

## Introduction
Rails provides a dedicated mode for building JSON APIs. The `--api` flag creates a leaner application that skips browser-specific middleware, views, and helpers. In Rails 8.1, this includes Solid Queue for background jobs and Solid Cache for caching by default.

## Key Concepts
- **API-Only Mode**: A Rails configuration that removes browser middleware (cookies, sessions, CSRF) and uses `ActionController::API` instead of `ActionController::Base`.
- **`ActionController::API`**: A lighter controller base class without view rendering, flash messages, or cookie handling.
- **Middleware Stack**: API-only apps include only middleware needed for JSON processing, logging, and security.
- **Solid Queue**: Rails 8's default background job backend, replacing the need for Redis-backed Sidekiq in many cases.

## Real World Context
Mobile apps, single-page applications, and microservices all need JSON APIs. Rails' API mode strips away HTML-specific features, reducing memory usage and response times. Companies like Shopify and GitHub run Rails API backends serving millions of requests.

## Deep Dive
### Creating the Application

```bash
rails new my_api --api
```

This generates an application with key differences from a standard Rails app:

```ruby
# config/application.rb
module MyApi
  class Application < Rails::Application
    config.load_defaults 8.1
    config.api_only = true
  end
end
```

The `config.api_only = true` setting configures the entire application for API mode. Controllers inherit from `ActionController::API`, the middleware stack is trimmed, and view generation is skipped.

### The API Controller Base

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  # No CSRF protection (APIs use token auth)
  # No session handling
  # No cookie handling
  # No flash messages
  # No view rendering helpers
end
```

`ActionController::API` includes only the modules needed for API responses: rendering, redirecting, parameter filtering, and basic callbacks. It's roughly 60% lighter than `ActionController::Base`.

### Rails 8.1 Defaults

New Rails 8.1 API apps include several modern defaults:

```ruby
# Gemfile (generated)
gem "solid_queue"  # Background jobs (replaces Redis + Sidekiq for many cases)
gem "solid_cache"  # SQL-based caching
gem "solid_cable"  # Action Cable without Redis
```

These "Solid" libraries use your existing database instead of requiring Redis, simplifying infrastructure for API applications.

## Common Pitfalls
1. **Adding browser middleware back unnecessarily** — If you need sessions or cookies, you probably want a full Rails app, not API-only.
2. **Forgetting CORS configuration** — API-only apps serving browser clients need `rack-cors` gem configured. Without it, browsers block cross-origin requests.

## Best Practices
1. **Start API-only and add back selectively** — It's easier to add specific middleware than to remove unnecessary ones.
2. **Use Solid Queue for background jobs** — It's the Rails 8 default and eliminates the Redis dependency for most job processing needs.

## Summary
- `rails new my_api --api` creates a lean API-focused application.
- API controllers inherit from `ActionController::API`, a lighter base class.
- Rails 8.1 includes Solid Queue, Solid Cache, and Solid Cable by default.
- API mode removes browser middleware (sessions, cookies, CSRF, flash).
- Configure `rack-cors` for browser clients making cross-origin requests.

## Code Examples

**The API-only application controller — inherits from ActionController::API with only API-essential modules**

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  # Lighter than ActionController::Base:
  # - No CSRF protection (use token auth instead)
  # - No session/cookie handling
  # - No flash messages or view helpers
  # - Includes: rendering, params, callbacks, rescue_from
end
```


## Resources

- [Using Rails for API-Only Applications](https://guides.rubyonrails.org/api_app.html) — Official guide on API-only mode, middleware, and configuration

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*