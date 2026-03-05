---
source_course: "rails-foundations"
source_lesson: "rails-foundations-model-testing"
---

# Testing Models

## Introduction

Model tests verify your business logic, validations, associations, and custom methods work correctly. They are the fastest tests to write and run, and they cover the core logic of your application.

## Key Concepts

- **Model test**: A test class inheriting from `ActiveSupport::TestCase` that tests a model's validations, methods, scopes, and associations.
- **Validation testing**: Verifying that models accept valid data and reject invalid data with appropriate error messages.
- **Association testing**: Confirming that model relationships (has_many, belongs_to) work correctly.
- **Scope testing**: Ensuring named scopes return the correct subset of records.

## Real World Context

Model tests are the foundation of your test suite. They run in milliseconds, require no browser, and test the logic that matters most. A well-tested model gives you confidence that your business rules are enforced regardless of how the data is accessed (web, API, console).

## Deep Dive

### Testing Validations

```ruby
class Article < ApplicationRecord
  validates :title, presence: true, length: { minimum: 5 }
  validates :body, presence: true
end
```

```ruby
class ArticleTest < ActiveSupport::TestCase
  test "valid with all attributes" do
    article = Article.new(title: "Hello World", body: "Content")
    assert article.valid?
  end

  test "invalid without title" do
    article = Article.new(body: "Content")
    assert_not article.valid?
    assert_includes article.errors[:title], "can't be blank"
  end

  test "invalid with short title" do
    article = Article.new(title: "Hi", body: "Content")
    assert_not article.valid?
  end
end
```

### Testing Associations

```ruby
class ArticleTest < ActiveSupport::TestCase
  test "has many comments" do
    article = articles(:published_article)
    assert_respond_to article, :comments
  end

  test "destroys comments when destroyed" do
    article = articles(:published_article)
    article.comments.create(body: "Test comment")
    assert_difference "Comment.count", -1 do
      article.destroy
    end
  end
end
```

### Testing Custom Methods

```ruby
class Article < ApplicationRecord
  def published?
    published_at.present? && published_at <= Time.current
  end

  def reading_time
    (body.split.size / 200.0).ceil
  end
end
```

```ruby
class ArticleTest < ActiveSupport::TestCase
  test "published? returns true for past dates" do
    article = Article.new(published_at: 1.day.ago)
    assert article.published?
  end

  test "reading_time calculates correctly" do
    article = Article.new(body: "word " * 400)
    assert_equal 2, article.reading_time
  end
end
```

### Testing Scopes

```ruby
class ArticleTest < ActiveSupport::TestCase
  test "published scope returns only published articles" do
    published = Article.create(title: "Pub", body: "x", published_at: 1.day.ago)
    draft = Article.create(title: "Draft", body: "x", published_at: nil)

    results = Article.published
    assert_includes results, published
    assert_not_includes results, draft
  end
end
```

## Common Pitfalls

- **Only testing the happy path**: Always test both valid and invalid cases. Test edge cases and boundary conditions.
- **Testing Rails itself**: Don't test that `validates :title, presence: true` works. Test your specific business rules and combinations.
- **Shared state between tests**: Each test should be independent. Use `setup` for shared setup and don't rely on test execution order.

## Best Practices

- Test validations by checking both valid and invalid states.
- Test custom methods with various inputs including edge cases.
- Use `assert_difference` to verify that create/destroy operations change record counts.

## Summary

- Model tests inherit from `ActiveSupport::TestCase` and live in `test/models/`.
- Test validations by creating invalid objects and checking `errors`.
- Test associations with `assert_respond_to` and `assert_difference` for dependent destroy.
- Test custom methods with representative inputs and edge cases.
- Model tests are fast and should be the largest portion of your test suite.

## Code Examples

**Model tests verify validations reject invalid data and associations behave correctly, including dependent destroy behavior.**

```ruby
class ArticleTest < ActiveSupport::TestCase
  test "invalid without title" do
    article = Article.new(body: "Content")
    assert_not article.valid?
    assert_includes article.errors[:title], "can't be blank"
  end

  test "destroys comments when destroyed" do
    article = articles(:published_article)
    article.comments.create(body: "Test")
    assert_difference "Comment.count", -1 do
      article.destroy
    end
  end
end
```


## Resources

- [Testing Rails Applications - Model Testing](https://guides.rubyonrails.org/testing.html#model-testing) — Guide to testing Rails models

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*