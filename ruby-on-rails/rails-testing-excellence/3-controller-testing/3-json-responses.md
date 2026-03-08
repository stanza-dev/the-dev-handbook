---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-json-responses"
---

# Testing JSON Responses

## Introduction

Modern Rails applications frequently expose JSON APIs consumed by single-page applications, mobile clients, or third-party integrations. Testing these endpoints requires verifying not just the status code but also the structure and content of the JSON payload. Rails ships with `response.parsed_body`, a convenient helper that automatically deserializes the response, making JSON assertions clean and readable.

## Key Concepts

- **response.parsed_body**: A Rails helper that automatically parses the response body based on the content type. For JSON responses it returns a Ruby hash or array, eliminating the need for manual `JSON.parse` calls.
- **as: :json**: A request option that sets the `Content-Type` and `Accept` headers to `application/json`, telling Rails to process the request and response as JSON.
- **JSON structure assertions**: Techniques for verifying that a JSON response contains the expected keys, values, and nested structures.
- **Error response testing**: Validating that invalid requests return proper error payloads with meaningful messages.

## Real World Context

When building an API, your clients depend on a stable contract: specific keys, data types, and error formats. If a developer accidentally renames a JSON key or removes a field, the mobile app or front-end client breaks silently. JSON response tests lock in that contract. They catch breaking changes at the test level before they reach production, acting as lightweight API contract tests.

## Deep Dive

Start with a basic test that verifies the response content type and parses the body. In Rails 8.1, prefer `response.parsed_body` over manual `JSON.parse(response.body)` — it is shorter and automatically adapts to the response content type:

```ruby
class Api::ArticlesControllerTest < ActionDispatch::IntegrationTest
  test 'index returns JSON array' do
    get api_articles_url, as: :json

    assert_response :success
    assert_equal 'application/json', response.media_type
    assert_kind_of Array, response.parsed_body
    assert_equal Article.count, response.parsed_body.size
  end
end
```

The `as: :json` option sets the appropriate headers so Rails routes the request through your API controller. `response.parsed_body` gives you a native Ruby object to assert against.

Next, test the structure of individual resource responses. Verify that expected fields are present and sensitive fields are absent:

```ruby
test 'show returns article with expected fields' do
  article = articles(:first)
  get api_article_url(article), as: :json

  json = response.parsed_body

  assert_equal article.id, json['id']
  assert_equal article.title, json['title']
  assert_equal article.body, json['body']
  assert json.key?('created_at')
  assert_not json.key?('password_digest')  # Sensitive field excluded
end
```

This pattern ensures your serializer includes the fields your clients need and excludes fields they should never see.

For creation endpoints, verify the status code and the returned object:

```ruby
test 'create returns 201 with the new article' do
  post api_articles_url,
    params: { article: { title: 'New', body: 'Content' } },
    as: :json

  assert_response :created

  json = response.parsed_body
  assert_equal 'New', json['title']
  assert json['id'].present?
end
```

The `:created` symbol maps to HTTP 201. Checking that an `id` is present confirms the record was persisted.

Finally, test error responses to ensure your API returns helpful error messages:

```ruby
test 'create returns errors for invalid data' do
  post api_articles_url,
    params: { article: { title: '' } },
    as: :json

  assert_response :unprocessable_entity

  json = response.parsed_body
  assert json['errors'].present?
  assert_includes json['errors']['title'], "can't be blank"
end

test 'show returns 404 for missing article' do
  get api_article_url(id: 999999), as: :json

  assert_response :not_found

  json = response.parsed_body
  assert_equal 'Article not found', json['error']
end
```

These tests document your API's error format, which clients rely on for displaying validation messages or handling missing resources.

## Common Pitfalls

1. **Using JSON.parse instead of parsed_body** — Manual parsing works but is verbose and does not benefit from Rails' automatic content-type detection. Prefer `response.parsed_body` in Rails 7.1+ for cleaner tests.
2. **Forgetting as: :json** — Without this option, Rails may process the request as HTML, returning a different response format or triggering CSRF protection. Always pass `as: :json` for API tests.
3. **Only testing the happy path** — An API that returns 200 for valid data but crashes on invalid input is incomplete. Always test error responses with missing or malformed parameters.

## Best Practices

1. **Assert on structure, not just values** — Check that keys exist and sensitive keys are absent. This catches serializer regressions even when fixture data changes.
2. **Use response.parsed_body consistently** — It works for JSON, HTML (returns a Nokogiri document), and other content types. Adopting it everywhere keeps your test style uniform.
3. **Test pagination metadata** — If your index endpoint paginates, assert on pagination keys (`total`, `page`, `per_page`) alongside the data array.

## Summary

- Use `as: :json` in test requests to set proper content-type headers for API endpoints.
- Prefer `response.parsed_body` over `JSON.parse(response.body)` for cleaner, Rails-native assertions.
- Verify both the presence of expected fields and the absence of sensitive fields in JSON responses.
- Test creation endpoints for 201 status and returned object structure.
- Always test error responses to document your API's error contract.

## Code Examples

**Testing a paginated JSON index, a successful create with 201, and validation error responses using response.parsed_body**

```ruby
class Api::ProductsControllerTest < ActionDispatch::IntegrationTest
  test 'index returns paginated JSON' do
    get api_products_url, as: :json

    assert_response :success
    body = response.parsed_body

    # Verify structure
    assert body.key?('products')
    assert body.key?('meta')
    assert_equal 1, body['meta']['page']
  end

  test 'create returns 201 with product data' do
    post api_products_url,
      params: { product: { name: 'Gadget', price: 29.99 } },
      as: :json

    assert_response :created
    assert_equal 'Gadget', response.parsed_body['name']
  end

  test 'create returns validation errors' do
    post api_products_url,
      params: { product: { name: '' } },
      as: :json

    assert_response :unprocessable_entity
    assert response.parsed_body['errors']['name'].present?
  end
end
```


## Resources

- [Rails Testing Guide — Testing JSON Responses](https://guides.rubyonrails.org/testing.html) — Official Rails testing guide with sections on testing API responses
- [Rails API Documentation — parsed_body](https://api.rubyonrails.org/classes/ActionDispatch/TestResponse.html) — API reference for the parsed_body helper on test responses

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*