---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-authentication"
---

# Testing Authentication

Most applications require authentication. Here's how to test protected actions.

## Testing Unauthenticated Access

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test 'redirects to login when not authenticated' do
    get new_article_url
    assert_redirected_to login_url
  end
  
  test 'returns unauthorized for API without token' do
    get api_articles_url
    assert_response :unauthorized
  end
end
```

## Helper for Logging In

Create a test helper method:

```ruby
# test/test_helper.rb
class ActionDispatch::IntegrationTest
  def sign_in(user)
    post login_url, params: {
      email: user.email,
      password: 'password'
    }
  end
  
  def sign_in_as(user)
    post session_url, params: {
      session: {
        email: user.email,
        password: 'password'
      }
    }
  end
end
```

## Testing Authenticated Actions

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  def setup
    @user = users(:david)
  end
  
  test 'can create article when logged in' do
    sign_in(@user)
    
    assert_difference('Article.count', 1) do
      post articles_url, params: {
        article: { title: 'New', body: 'Content' }
      }
    end
  end
  
  test 'can only edit own articles' do
    sign_in(@user)
    other_article = articles(:by_other_user)
    
    patch article_url(other_article), params: {
      article: { title: 'Hacked' }
    }
    
    assert_response :forbidden
  end
end
```

## Testing API Authentication

```ruby
class Api::ArticlesControllerTest < ActionDispatch::IntegrationTest
  def setup
    @user = users(:david)
    @token = @user.generate_api_token
  end
  
  test 'can access with valid token' do
    get api_articles_url, headers: {
      'Authorization' => "Bearer #{@token}"
    }
    assert_response :success
  end
end
```

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*