---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-callbacks"
---

# Testing Callbacks and Methods

## Introduction

Callbacks contain business logic that runs automatically at specific points in a model's lifecycle — normalizing data before save, sending emails after create, or updating counters after destroy. Custom methods and scopes also encode important behavior. Testing these ensures your models do what they promise without manual intervention.

## Key Concepts

- **before_save / before_validation**: Callbacks that transform data before it reaches the database. Test by saving a record and inspecting the resulting attributes.
- **after_create**: Callbacks that trigger side effects when a new record is created. Test with `assert_enqueued_emails` or `assert_difference` for counter caches.
- **Scopes**: Named queries defined on the model. Test by verifying that the returned collection includes expected records and excludes others.
- **Custom Methods**: Instance or class methods that encapsulate business logic. Test like any other Ruby method with assertions on return values and side effects.

## Real World Context

A `before_save` callback that normalizes email addresses to lowercase is invisible to developers calling `User.create!`. Without a test, someone could refactor the callback and accidentally break it, letting mixed-case emails into the database and causing login failures. Callback tests make implicit behavior explicit and prevent silent regressions.

## Deep Dive

### Testing before_save Callbacks

Test the outcome of the callback by saving a record and inspecting the persisted value:

```ruby
class UserTest < ActiveSupport::TestCase
  test 'normalizes email to lowercase before save' do
    user = User.create!(
      name: 'Alice',
      email: '  ALICE@EXAMPLE.COM  '
    )

    assert_equal 'alice@example.com', user.email
  end
end
```

Notice that the test provides a mixed-case email with extra whitespace and asserts the saved result is clean. This tests the full callback chain without coupling to implementation details.

### Testing after_create Callbacks

When a callback sends an email or enqueues a job, use Rails' built-in test helpers:

```ruby
class OrderTest < ActiveSupport::TestCase
  test 'sends confirmation email after creation' do
    assert_enqueued_emails 1 do
      Order.create!(
        customer: 'Alice',
        total: 99.99
      )
    end
  end
end
```

For counter caches or other database side effects:

```ruby
test 'increments author article count after create' do
  author = authors(:david)

  assert_difference 'author.reload.articles_count', 1 do
    author.articles.create!(
      title: 'New Article',
      body: 'Content'
    )
  end
end
```

### Testing Scopes

Scopes are testable queries. Verify inclusion and exclusion:

```ruby
class ArticleTest < ActiveSupport::TestCase
  test 'published scope returns only published articles' do
    published = articles(:published_one)
    draft = articles(:draft)

    results = Article.published

    assert_includes results, published
    assert_not_includes results, draft
  end

  test 'recent scope returns articles from the last week' do
    recent = articles(:today)
    old = articles(:last_month)

    results = Article.recent

    assert_includes results, recent
    assert_not_includes results, old
  end
end
```

### Testing Custom Instance Methods

Test custom methods like any Ruby method — call them and assert on the return value:

```ruby
class UserTest < ActiveSupport::TestCase
  test 'full_name joins first and last name' do
    user = User.new(first_name: 'John', last_name: 'Doe')
    assert_equal 'John Doe', user.full_name
  end

  test 'full_name handles missing last name' do
    user = User.new(first_name: 'John', last_name: nil)
    assert_equal 'John', user.full_name
  end

  test 'adult? returns true for age 18 and above' do
    assert User.new(age: 18).adult?
    assert User.new(age: 25).adult?
  end

  test 'adult? returns false for under 18 or nil' do
    assert_not User.new(age: 17).adult?
    assert_not User.new(age: nil).adult?
  end
end
```

## Common Pitfalls

1. **Testing the callback implementation instead of its effect** — Do not test that `normalize_email` is called. Test that the saved email is lowercase. This lets you refactor the implementation freely.
2. **Forgetting time-dependent tests** — Scopes like `recent` depend on the current time. Use `travel_to` to freeze time and make tests deterministic: `travel_to Time.zone.parse('2025-01-15') do ... end`.

## Best Practices

1. **Use travel_to for time-sensitive tests** — Rails provides `travel_to` to freeze or travel through time. Always use it when testing scopes or callbacks that depend on timestamps.
2. **Test edge cases for custom methods** — If `full_name` concatenates first and last name, test what happens when one or both are nil, empty, or contain special characters.

## Summary

- Test callbacks by verifying their effect on persisted data, not by checking that the callback method was called.
- Use `assert_enqueued_emails` and `assert_difference` to test after_create side effects.
- Test scopes by verifying both inclusion of matching records and exclusion of non-matching records.
- Use `travel_to` for deterministic time-dependent tests.

## Code Examples

**Testing a before_save callback by checking persisted data and a scope by verifying inclusion and exclusion**

```ruby
class UserTest < ActiveSupport::TestCase
  test 'normalizes email before save' do
    user = User.create!(
      name: 'Alice',
      email: '  ALICE@EXAMPLE.COM  '
    )
    # The before_save callback strips and downcases
    assert_equal 'alice@example.com', user.email
  end
end

class ArticleTest < ActiveSupport::TestCase
  test 'published scope excludes drafts' do
    results = Article.published
    assert_includes results, articles(:published_one)
    assert_not_includes results, articles(:draft)
  end
end
```


## Resources

- [Active Record Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html) — Official guide to all callback types and their execution order
- [Active Record Query Interface - Scopes](https://guides.rubyonrails.org/active_record_querying.html#scopes) — Official guide on defining and using model scopes

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*