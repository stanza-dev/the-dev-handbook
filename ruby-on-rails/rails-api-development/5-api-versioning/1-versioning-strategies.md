---
source_course: "rails-api-development"
source_lesson: "rails-api-development-versioning-strategies"
---

# API Versioning Strategies

## Introduction
As APIs evolve, you need to make breaking changes without disrupting existing clients. API versioning lets you run multiple versions simultaneously, giving clients time to migrate.

## Key Concepts
- **URL Versioning**: Version in the URL path (`/api/v1/products`). Most common and explicit.
- **Header Versioning**: Version in a custom header (`Api-Version: 2`). Cleaner URLs but less discoverable.
- **Namespace Versioning**: Using Rails modules to organize versioned controllers.
- **Deprecation**: Marking old versions as deprecated before removal.

## Real World Context
GitHub uses URL versioning, Stripe uses API version dates in headers, and Twilio uses URL versioning. URL versioning is the most common because it's explicit, easy to test, and works with all HTTP clients without special header configuration.

## Deep Dive
### URL Versioning with Namespaces

```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :products
      resources :users
    end

    namespace :v2 do
      resources :products
      resources :users
    end
  end
end
```

Controllers live in separate modules:

```ruby
# app/controllers/api/v1/products_controller.rb
module Api::V1
  class ProductsController < ApplicationController
    def index
      render json: Product.all.as_json(only: [:id, :name, :price])
    end
  end
end

# app/controllers/api/v2/products_controller.rb
module Api::V2
  class ProductsController < ApplicationController
    def index
      render json: ProductSerializer.new(Product.all).serializable_hash
    end
  end
end
```

V1 uses simple as_json, V2 uses a serializer with a richer format. Both run simultaneously.

### Sharing Logic Between Versions

```ruby
# app/controllers/concerns/product_actions.rb
module ProductActions
  extend ActiveSupport::Concern

  def set_product
    @product = Product.find(params[:id])
  end

  def product_params
    params.require(:product).permit(:name, :price, :description)
  end
end

# Both versions include shared logic
module Api::V1
  class ProductsController < ApplicationController
    include ProductActions
  end
end
```

Concerns prevent code duplication between versions.

## Common Pitfalls
1. **Not versioning from day one** — Adding versioning later requires restructuring all controllers and routes.
2. **Too many active versions** — Maintain at most 2-3 versions. Deprecate and remove old ones.

## Best Practices
1. **Start with `/api/v1/` from the beginning** — Even if you think you won't need v2.
2. **Share logic via concerns** — Don't duplicate business logic across versions.

## Summary
- URL versioning (`/api/v1/`) is the most common and explicit strategy.
- Rails namespaces map cleanly to versioned controller modules.
- Share common logic between versions using concerns.
- Deprecate old versions before removing them.
- Start versioning from day one.

## Code Examples

**Namespaced routes supporting multiple API versions running simultaneously**

```ruby
# config/routes.rb — Multi-version API routing
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :products
    end
    namespace :v2 do
      resources :products
    end
  end
end
# /api/v1/products → Api::V1::ProductsController
# /api/v2/products → Api::V2::ProductsController
```


## Resources

- [Rails Routing Namespaces](https://guides.rubyonrails.org/routing.html#controller-namespaces-and-routing) — Guide on namespace routing for API versioning

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*