---
source_course: "rails-foundations"
source_lesson: "rails-foundations-controller-testing"
---

# Testing Controllers

Controller tests (or request tests) verify that your controllers handle HTTP requests correctly.

## Controller Test Basics

```ruby
# test/controllers/articles_controller_test.rb
require "test_helper"

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

## HTTP Methods

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get articles_url
    assert_response :success
  end

  test "should get new" do
    get new_article_url
    assert_response :success
  end

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
end
```

## Testing Responses

```ruby
# Response status
assert_response :success        # 200
assert_response :redirect       # 3xx
assert_response :not_found      # 404
assert_response :unprocessable_entity  # 422
assert_response 201             # Specific code

# Redirects
assert_redirected_to articles_url
assert_redirected_to @article
assert_redirected_to root_path

# Response body
assert_match "Article Title", response.body
assert_select "h1", "Article Title"  # CSS selector
assert_select "article", count: 3
```

## Testing with Authentication

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  setup do
    @user = users(:one)
    @article = articles(:one)
  end

  test "should redirect create when not logged in" do
    post articles_url, params: { article: { title: "Test" } }
    assert_redirected_to login_url
  end

  test "should create article when logged in" do
    # Log in the user
    post login_url, params: { email: @user.email, password: "password" }

    assert_difference("Article.count") do
      post articles_url, params: {
        article: { title: "New", body: "Content" }
      }
    end
  end
end
```

## Testing Invalid Data

```ruby
test "should not create article with invalid data" do
  assert_no_difference("Article.count") do
    post articles_url, params: {
      article: { title: "", body: "" }  # Invalid
    }
  end

  assert_response :unprocessable_entity
end

test "should show validation errors" do
  post articles_url, params: { article: { title: "" } }

  assert_select ".errors" do
    assert_select "li", /Title can't be blank/
  end
end
```

## Testing JSON APIs

```ruby
test "should return articles as JSON" do
  get articles_url, as: :json

  assert_response :success
  json = JSON.parse(response.body)
  assert_kind_of Array, json
end

test "should create article via API" do
  post articles_url,
    params: { article: { title: "API Article", body: "Content" } },
    as: :json

  assert_response :created
  json = JSON.parse(response.body)
  assert_equal "API Article", json["title"]
end
```

## Resources

- [Testing Rails Applications - Integration Testing](https://guides.rubyonrails.org/testing.html#integration-testing) — Guide to integration and controller testing

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*