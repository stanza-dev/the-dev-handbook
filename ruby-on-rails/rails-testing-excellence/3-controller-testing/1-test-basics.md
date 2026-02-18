---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-controller-test-basics"
---

# Controller Test Basics

Controller tests verify that your actions respond correctly to requests. Rails 5+ uses integration-style controller tests.

## Basic Controller Test

```ruby
require 'test_helper'

class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test 'should get index' do
    get articles_url
    assert_response :success
  end
  
  test 'should get show' do
    article = articles(:first)
    get article_url(article)
    assert_response :success
  end
  
  test 'should get new' do
    get new_article_url
    assert_response :success
  end
end
```

## HTTP Methods

Test different HTTP methods:

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test 'should create article' do
    assert_difference('Article.count', 1) do
      post articles_url, params: {
        article: { title: 'New', body: 'Content' }
      }
    end
    assert_redirected_to article_url(Article.last)
  end
  
  test 'should update article' do
    article = articles(:first)
    patch article_url(article), params: {
      article: { title: 'Updated Title' }
    }
    assert_redirected_to article_url(article)
    article.reload
    assert_equal 'Updated Title', article.title
  end
  
  test 'should destroy article' do
    article = articles(:first)
    assert_difference('Article.count', -1) do
      delete article_url(article)
    end
    assert_redirected_to articles_url
  end
end
```

## Response Assertions

```ruby
# Check response status
assert_response :success      # 200-299
assert_response :redirect     # 300-399
assert_response :not_found    # 404
assert_response 201           # Specific status code

# Check redirects
assert_redirected_to articles_url
assert_redirected_to @article

# Check response body
assert_match 'Welcome', response.body
```

See [Testing Controllers](https://guides.rubyonrails.org/testing.html#functional-tests-for-your-controllers).

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*