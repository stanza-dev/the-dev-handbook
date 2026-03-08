---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-tdd-feature-development"
---

# TDD for Feature Development

## Introduction
Model TDD verifies business logic in isolation, but real features span controllers, views, and routes. Feature-level TDD uses integration and system tests to drive the implementation of entire user-facing workflows — from visiting a URL to seeing the result on screen. This outside-in approach starts with the user experience and works inward, implementing each layer only when a test demands it.

## Key Concepts
- **Outside-In TDD**: A strategy where you start with a high-level test (system or integration) that describes the full user journey, then drop down to unit tests for specific model logic as needed.
- **Integration Test**: A test that sends HTTP requests to your application and asserts on the response — status codes, redirects, and response body — without a browser.
- **System Test**: A browser-driven test that interacts with the full rendered page, including JavaScript.
- **Double Loop TDD**: The practice of writing an outer failing system test, then implementing the feature through a series of inner Red-Green-Refactor cycles at the model and controller level.

## Real World Context
Imagine your product manager says: "Users should be able to publish a draft article, and published articles should appear on the public index." With outside-in TDD, you write a system test that captures this entire scenario — visit the draft article, click Publish, verify it appears on the index. Then you implement the route, controller action, model method, and view changes, each driven by a failing test.

## Deep Dive

### Step 1: Write the Outer System Test
Start with a system test that describes the complete user journey:

```ruby
# test/system/publishing_test.rb
require 'application_system_test_case'

class PublishingTest < ApplicationSystemTestCase
  test 'author publishes a draft article' do
    article = articles(:draft)
    sign_in users(:author)

    visit article_url(article)
    assert_text 'Draft'

    click_on 'Publish'

    assert_text 'Article was successfully published'
    assert_no_text 'Draft'

    visit articles_url
    assert_selector '.article', text: article.title
  end
end
```

This test fails immediately because the "Publish" button does not exist yet. That failure drives your next steps.

### Step 2: Add the Route and Controller Action
The system test needs a `publish` action. Write a controller test first:

```ruby
# test/controllers/articles_controller_test.rb
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test 'publish transitions article to published state' do
    article = articles(:draft)
    sign_in users(:author)

    patch publish_article_url(article)

    assert_redirected_to article_url(article)
    assert article.reload.published?
    assert_equal 'Article was successfully published', flash[:notice]
  end

  test 'publish requires authentication' do
    article = articles(:draft)

    patch publish_article_url(article)

    assert_redirected_to login_url
  end
end
```

This fails because the route and action do not exist. Add them:

```ruby
# config/routes.rb
resources :articles do
  member do
    patch :publish
  end
end
```

```ruby
# app/controllers/articles_controller.rb
def publish
  @article = Article.find(params[:id])
  @article.publish!
  redirect_to @article, notice: 'Article was successfully published'
end
```

### Step 3: Implement the Model Method via TDD
The controller calls `publish!`, which does not exist yet. Drop down to a model test:

```ruby
# test/models/article_test.rb
class ArticleTest < ActiveSupport::TestCase
  test 'publish! sets published_at to current time' do
    article = articles(:draft)
    assert_nil article.published_at

    travel_to Time.zone.local(2026, 3, 8) do
      article.publish!
      assert_equal Time.zone.local(2026, 3, 8), article.published_at
    end
  end

  test 'published? returns true when published_at is set' do
    article = Article.new(published_at: 1.hour.ago)
    assert article.published?
  end

  test 'draft? returns true when published_at is nil' do
    article = Article.new(published_at: nil)
    assert article.draft?
  end
end
```

Implement the model methods:

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  def publish!
    update!(published_at: Time.current)
  end

  def published?
    published_at.present?
  end

  def draft?
    !published?
  end
end
```

### Step 4: Add the View Button
The model and controller are done, but the system test still fails because the "Publish" button is not in the view. Add it:

```erb
<%# app/views/articles/show.html.erb %>
<% if @article.draft? %>
  <span class="badge">Draft</span>
  <%= button_to 'Publish', publish_article_path(@article), method: :patch %>
<% end %>
```

Run the outer system test again. It passes. The entire feature was driven by tests at every layer.

### The Double Loop Visualized
The workflow forms two nested loops:

1. **Outer loop**: System test (Red) drives the feature. You keep returning to this test to check if the full journey works.
2. **Inner loops**: Controller tests and model tests (Red-Green-Refactor) implement each piece. Once all inner tests pass, the outer test passes too.

## Common Pitfalls
1. **Starting with the model instead of the user journey** — Outside-in TDD starts with the system test. If you start with model tests, you may build methods nobody calls or miss the controller glue.
2. **Making the outer test pass in one big step** — Break the work into inner Red-Green-Refactor cycles. Each layer (route, controller, model, view) gets its own test.
3. **Skipping the controller test** — Even with a system test, controller tests are faster and more precise. They catch authorization bugs and redirect logic that system tests verify only indirectly.

## Best Practices
1. **Start with the system test, implement from the inside out** — Write the outer test first, then implement model, controller, and view, each with their own tests.
2. **Keep the outer test as a checkpoint** — Run the system test after each inner cycle. When it goes green, the feature is done.
3. **Use `bin/rails test` for fast inner loops** — Model and controller tests run in milliseconds. Save the slower system test for the final verification.

## Summary
- Outside-in TDD starts with a system test that describes the full user journey.
- Drop down to controller and model tests for each layer of implementation.
- The double loop pattern uses the system test as the outer checkpoint and model/controller tests as inner Red-Green-Refactor cycles.
- Each layer is implemented only when a failing test demands it.
- Rails 8 generators create both the implementation file and its corresponding test file, supporting TDD from the start.

## Code Examples

**The outside-in TDD workflow — the system test drives implementation of routes, controllers, models, and views**

```ruby
# Outside-in TDD: Start with the system test
class PublishingTest < ApplicationSystemTestCase
  test 'author publishes a draft article' do
    article = articles(:draft)
    sign_in users(:author)

    visit article_url(article)
    click_on 'Publish'

    assert_text 'Article was successfully published'
    visit articles_url
    assert_selector '.article', text: article.title
  end
end

# Then implement the inner layers:
# 1. Route: patch :publish on articles
# 2. Controller: ArticlesController#publish
# 3. Model: Article#publish! and Article#published?
# 4. View: Publish button in show.html.erb
```


## Resources

- [Rails Testing Guide](https://guides.rubyonrails.org/testing.html) — Official Rails testing guide covering integration tests, system tests, and the full testing stack

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*