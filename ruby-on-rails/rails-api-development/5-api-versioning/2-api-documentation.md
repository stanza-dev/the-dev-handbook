---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-documentation"
---

# API Documentation

## Introduction
Good documentation is essential for API adoption. Rails APIs can be documented with OpenAPI/Swagger specifications that generate interactive documentation, client SDKs, and validation.

## Key Concepts
- **OpenAPI (Swagger)**: A specification for describing REST APIs in a machine-readable format.
- **rswag**: A Rails gem that generates OpenAPI specs from RSpec tests.
- **Swagger UI**: An interactive documentation interface generated from OpenAPI specs.
- **API Blueprint**: An alternative documentation format using Markdown.

## Real World Context
APIs without documentation are APIs nobody uses. Interactive documentation lets developers try endpoints directly from the docs. Generated specs ensure documentation stays in sync with the actual API.

## Deep Dive
### Using rswag

```ruby
# Gemfile
gem "rswag-api"
gem "rswag-ui"
gem "rswag-specs", group: :test
```

Write specs that double as documentation:

```ruby
# spec/requests/api/v1/products_spec.rb
RSpec.describe "Products API", type: :request do
  path "/api/v1/products" do
    get "List products" do
      tags "Products"
      produces "application/json"
      parameter name: :page, in: :query, type: :integer, required: false
      parameter name: :per_page, in: :query, type: :integer, required: false

      response "200", "products found" do
        schema type: :object, properties: {
          data: { type: :array, items: { "$ref" => "#/components/schemas/product" } },
          meta: { type: :object, properties: { total: { type: :integer } } }
        }
        run_test!
      end
    end
  end
end
```

Running `rails rswag:specs:swaggerize` generates the OpenAPI JSON spec. Mount Swagger UI to serve interactive docs:

```ruby
# config/routes.rb
mount Rswag::Ui::Engine => "/api-docs"
mount Rswag::Api::Engine => "/api-docs"
```

Now `/api-docs` shows interactive documentation where developers can try each endpoint.

## Common Pitfalls
1. **Documentation drift** — Manual docs get out of sync. Use rswag or similar tools that generate docs from tests.
2. **Missing error responses** — Document error cases (400, 401, 404, 422) not just success cases.

## Best Practices
1. **Generate docs from tests** — rswag ensures docs match the actual API behavior.
2. **Include authentication instructions** — Document how to obtain and use API tokens.

## Summary
- OpenAPI/Swagger is the standard for REST API documentation.
- rswag generates OpenAPI specs from RSpec tests, preventing documentation drift.
- Swagger UI provides interactive documentation at `/api-docs`.
- Document error responses, authentication, and pagination, not just happy paths.

## Code Examples

**rswag spec that tests the endpoint AND generates OpenAPI documentation simultaneously**

```ruby
# spec/requests/api/v1/products_spec.rb
RSpec.describe "Products API", type: :request do
  path "/api/v1/products/{id}" do
    get "Get a product" do
      tags "Products"
      produces "application/json"
      parameter name: :id, in: :path, type: :integer

      response "200", "product found" do
        let(:id) { create(:product).id }
        run_test!
      end

      response "404", "product not found" do
        let(:id) { 999999 }
        run_test!
      end
    end
  end
end
```


## Resources

- [rswag](https://github.com/rswag/rswag) — rswag gem for generating Swagger/OpenAPI docs from RSpec tests

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*