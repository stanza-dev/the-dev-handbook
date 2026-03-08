---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-routing"
---

# API Controllers and Routing

## Introduction
API controllers follow RESTful conventions, responding with JSON instead of HTML. Rails routing provides powerful tools for building clean, versioned API URLs with proper HTTP verb semantics.

## Key Concepts
- **RESTful Resources**: Routes that map HTTP verbs (GET, POST, PUT, DELETE) to controller actions (index, show, create, update, destroy).
- **Namespace**: URL prefix grouping (e.g., `/api/v1/`) that organizes controllers into modules.
- **`render json:`**: The primary way to return JSON responses from controllers.
- **Strong Parameters**: `params.require(:model).permit(:field)` for safe parameter filtering.

## Real World Context
Every production API needs clean URL structure, proper HTTP semantics, and organized code. API versioning through namespaces lets you evolve your API without breaking existing clients.

## Deep Dive
### Namespaced API Routes

```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :products, only: [:index, :show, :create, :update, :destroy]
      resources :users, only: [:show, :create, :update]
      resources :orders, only: [:index, :show, :create]
    end
  end
end
```

This generates URLs like `/api/v1/products` and expects controllers in `app/controllers/api/v1/`.

### API Controller Pattern

```ruby
# app/controllers/api/v1/products_controller.rb
module Api
  module V1
    class ProductsController < ApplicationController
      def index
        products = Product.all
        render json: products
      end

      def show
        product = Product.find(params[:id])
        render json: product
      end

      def create
        product = Product.new(product_params)
        if product.save
          render json: product, status: :created
        else
          render json: { errors: product.errors.full_messages }, status: :unprocessable_entity
        end
      end

      def update
        product = Product.find(params[:id])
        if product.update(product_params)
          render json: product
        else
          render json: { errors: product.errors.full_messages }, status: :unprocessable_entity
        end
      end

      def destroy
        product = Product.find(params[:id])
        product.destroy
        head :no_content
      end

      private

      def product_params
        params.require(:product).permit(:name, :price, :description)
      end
    end
  end
end
```

Each action follows REST semantics: `index` lists all, `show` returns one, `create` makes a new record (201 Created), `update` modifies (200 OK), `destroy` removes (204 No Content).

### HTTP Status Codes

Use appropriate status codes in responses:

```ruby
render json: product, status: :ok              # 200
render json: product, status: :created         # 201
head :no_content                                # 204
render json: { error: "Not found" }, status: :not_found  # 404
render json: { errors: errors }, status: :unprocessable_entity  # 422
```

Status codes communicate the result to API clients. They're essential for proper error handling on the client side.

## Common Pitfalls
1. **Returning 200 for everything** — Different outcomes need different status codes. Created resources get 201, deletions get 204, validation failures get 422.
2. **Not using namespaces** — Without `/api/v1/` namespacing, you can't version your API later without breaking clients.

## Best Practices
1. **Always namespace under `/api/v1/`** — Plan for versioning from day one.
2. **Use `head :no_content` for destroy** — Don't return the deleted object; return 204.

## Summary
- Namespace routes under `api/v1` for versioned, organized APIs.
- Controllers in `Api::V1` module respond with `render json:`.
- Use correct HTTP status codes: 200 OK, 201 Created, 204 No Content, 422 Unprocessable Entity.
- Strong parameters filter allowed fields with `params.require().permit()`.
- RESTful actions map HTTP verbs to controller methods.

## Code Examples

**Namespaced API routes generating clean versioned URLs with RESTful actions**

```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :products
      resources :users, only: [:show, :create, :update]
    end
  end
end
# Generates:
# GET    /api/v1/products     => api/v1/products#index
# POST   /api/v1/products     => api/v1/products#create
# GET    /api/v1/products/:id => api/v1/products#show
# PATCH  /api/v1/products/:id => api/v1/products#update
# DELETE /api/v1/products/:id => api/v1/products#destroy
```


## Resources

- [Rails Routing Guide](https://guides.rubyonrails.org/routing.html) — Complete guide on Rails routing including namespaces and resources

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*