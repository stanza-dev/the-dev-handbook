---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-documentation"
---

# API Documentation with OpenAPI

Good documentation is essential for API adoption. OpenAPI (formerly Swagger) is the industry standard.

## OpenAPI Specification

OpenAPI documents your API in YAML or JSON:

```yaml
# openapi.yaml
openapi: 3.0.3
info:
  title: My API
  version: 1.0.0
  description: A sample API

servers:
  - url: https://api.example.com/v1

paths:
  /articles:
    get:
      summary: List articles
      tags: [Articles]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
        - name: published
          in: query
          schema:
            type: boolean
      responses:
        200:
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Article'

    post:
      summary: Create article
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ArticleInput'
      responses:
        201:
          description: Created
        422:
          description: Validation error

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer

  schemas:
    Article:
      type: object
      properties:
        id:
          type: integer
        title:
          type: string
        body:
          type: string
        published:
          type: boolean
        created_at:
          type: string
          format: date-time

    ArticleInput:
      type: object
      required: [title, body]
      properties:
        title:
          type: string
          minLength: 1
          maxLength: 200
        body:
          type: string
          minLength: 10
```

## Using rswag Gem

Generate OpenAPI docs from RSpec tests:

```ruby
# Gemfile
gem 'rswag-api'
gem 'rswag-ui'
group :test do
  gem 'rswag-specs'
end
```

```ruby
# spec/requests/api/v1/articles_spec.rb
require 'swagger_helper'

RSpec.describe 'Articles API', type: :request do
  path '/api/v1/articles' do
    get 'List articles' do
      tags 'Articles'
      produces 'application/json'

      parameter name: :page, in: :query, type: :integer, required: false
      parameter name: :published, in: :query, type: :boolean, required: false

      response '200', 'articles found' do
        schema type: :object,
               properties: {
                 data: {
                   type: :array,
                   items: { '$ref' => '#/components/schemas/Article' }
                 }
               }

        run_test!
      end
    end

    post 'Create article' do
      tags 'Articles'
      consumes 'application/json'
      produces 'application/json'
      security [bearerAuth: []]

      parameter name: :article, in: :body, schema: {
        type: :object,
        properties: {
          title: { type: :string },
          body: { type: :string }
        },
        required: %w[title body]
      }

      response '201', 'article created' do
        let(:Authorization) { "Bearer #{user_token}" }
        let(:article) { { title: 'Test', body: 'Content' } }
        run_test!
      end

      response '422', 'invalid request' do
        let(:Authorization) { "Bearer #{user_token}" }
        let(:article) { { title: '' } }
        run_test!
      end
    end
  end
end
```

Generate docs:
```bash
RAILS_ENV=test rake rswag:specs:swaggerize
```

## Swagger UI

Serve interactive documentation:

```ruby
# config/routes.rb
mount Rswag::Ui::Engine => '/api-docs'
mount Rswag::Api::Engine => '/api-docs'
```

Visit `/api-docs` for interactive documentation!

## Resources

- [rswag Gem](https://github.com/rswag/rswag) — OpenAPI documentation for Rails APIs

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*