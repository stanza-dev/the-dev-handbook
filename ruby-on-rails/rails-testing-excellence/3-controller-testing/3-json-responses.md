---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-json-responses"
---

# Testing JSON Responses

For API controllers, you need to verify JSON response structure and content.

## Basic JSON Response Testing

```ruby
class Api::ArticlesControllerTest < ActionDispatch::IntegrationTest
  test 'index returns JSON' do
    get api_articles_url, as: :json
    
    assert_response :success
    assert_equal 'application/json', response.media_type
  end
  
  test 'index returns array of articles' do
    get api_articles_url, as: :json
    
    json = JSON.parse(response.body)
    assert_kind_of Array, json
    assert_equal Article.count, json.size
  end
end
```

## Testing JSON Structure

```ruby
class Api::ArticlesControllerTest < ActionDispatch::IntegrationTest
  test 'show returns article with expected fields' do
    article = articles(:first)
    get api_article_url(article), as: :json
    
    json = JSON.parse(response.body)
    
    assert_equal article.id, json['id']
    assert_equal article.title, json['title']
    assert_equal article.body, json['body']
    assert json.key?('created_at')
    assert_not json.key?('password_digest')  # Sensitive field
  end
  
  test 'create returns created article' do
    post api_articles_url,
      params: { article: { title: 'New', body: 'Content' } },
      as: :json
    
    assert_response :created
    
    json = JSON.parse(response.body)
    assert_equal 'New', json['title']
    assert json['id'].present?
  end
end
```

## Testing Error Responses

```ruby
class Api::ArticlesControllerTest < ActionDispatch::IntegrationTest
  test 'create returns errors for invalid data' do
    post api_articles_url,
      params: { article: { title: '' } },
      as: :json
    
    assert_response :unprocessable_entity
    
    json = JSON.parse(response.body)
    assert json['errors'].present?
    assert_includes json['errors']['title'], "can't be blank"
  end
  
  test 'show returns 404 for missing article' do
    get api_article_url(id: 999999), as: :json
    
    assert_response :not_found
    
    json = JSON.parse(response.body)
    assert_equal 'Article not found', json['error']
  end
end
```

## Using response.parsed_body

Rails provides a convenient helper:

```ruby
test 'can use parsed_body helper' do
  get api_articles_url, as: :json
  
  # Automatically parses JSON
  assert_equal Article.count, response.parsed_body.size
end
```

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*