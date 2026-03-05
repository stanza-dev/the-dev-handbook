---
source_course: "rails-foundations"
source_lesson: "rails-foundations-controller-testing"
---

# Testing Controllers

## Introduction

Controller tests (integration tests in modern Rails) verify that your application handles HTTP requests correctly, returns proper status codes, and renders the right templates.

## Key Concepts

- **Integration test**: A test that simulates HTTP requests (GET, POST, PATCH, DELETE) against your application.
- **`assert_response`**: Verifies the HTTP status code of the response (`:success`, `:redirect`, `:not_found`).
- **`assert_redirected_to`**: Verifies that the response redirected to a specific URL.
- **`assert_difference`**: Commonly used to verify that POST/DELETE actions change record counts.

## Real World Context

Controller tests verify the full request cycle: routing, authentication, parameter handling, and response. They catch issues like broken routes, missing authentication checks, and incorrect redirects that model tests cannot detect.

## Deep Dive

### Basic Controller Tests

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  setup do
    @article = articles(:published_article)
  end

  test "should get index" do
    get articles_url
    assert_response :success
  end

  test "should get show" do
    get article_url(@article)
    assert_response :success
  end
end
```

### Testing CRUD Operations

```ruby
test "should create article" do
  assert_difference("Article.count") do
    post articles_url, params: {
      article: { title: "New Article", body: "Content here" }
    }
  end
  assert_redirected_to article_url(Article.last)
end

test "should update article" do
  patch article_url(@article), params: {
    article: { title: "Updated Title" }
  }
  assert_redirected_to article_url(@article)
  @article.reload
  assert_equal "Updated Title", @article.title
end

test "should destroy article" do
  assert_difference("Article.count", -1) do
    delete article_url(@article)
  end
  assert_redirected_to articles_url
end
```

### Testing Invalid Data

```ruby
test "should not create article with invalid data" do
  assert_no_difference("Article.count") do
    post articles_url, params: {
      article: { title: "", body: "" }
    }
  end
  assert_response :unprocessable_entity
end
```

### Testing with Authentication

```ruby
test "should redirect create when not logged in" do
  post articles_url, params: { article: { title: "Test" } }
  assert_redirected_to login_url
end

test "should create article when logged in" do
  post login_url, params: { email: @user.email, password: "password" }
  assert_difference("Article.count") do
    post articles_url, params: { article: { title: "New", body: "Content" } }
  end
end
```

### Testing JSON APIs

```ruby
test "should return articles as JSON" do
  get articles_url, as: :json
  assert_response :success
  json = JSON.parse(response.body)
  assert_kind_of Array, json
end
```

## Common Pitfalls

- **Forgetting `.reload`**: After a PATCH request, the local variable still holds old data. Call `@article.reload` before asserting updated values.
- **Not testing unauthorized access**: Always test that protected actions redirect unauthenticated users.
- **Skipping negative tests**: Test that invalid data returns `:unprocessable_entity` and does not create records.

## Best Practices

- Test both authenticated and unauthenticated access for protected actions.
- Use `assert_difference` and `assert_no_difference` to verify data changes.
- Always check both the response status and any side effects (redirects, database changes).

## Summary

- Controller tests use `get`, `post`, `patch`, `delete` to simulate HTTP requests.
- `assert_response :success` checks for 200-range status codes.
- `assert_redirected_to` verifies redirect destinations.
- `assert_difference` confirms that create/destroy operations modify the database.
- Test both valid and invalid inputs, and both authenticated and unauthenticated access.

## Code Examples

**Integration tests simulate HTTP requests and verify responses, status codes, redirects, and database side effects.**

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get articles_url
    assert_response :success
  end

  test "should create article" do
    assert_difference("Article.count") do
      post articles_url, params: {
        article: { title: "New", body: "Content" }
      }
    end
    assert_redirected_to article_url(Article.last)
  end
end
```


## Resources

- [Testing Rails Applications - Integration Testing](https://guides.rubyonrails.org/testing.html#integration-testing) — Guide to integration and controller testing

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*