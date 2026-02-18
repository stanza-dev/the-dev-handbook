---
source_course: "rails-foundations"
source_lesson: "rails-foundations-testing-introduction"
---

# Introduction to Rails Testing

Testing ensures your application works correctly and continues to work as you make changes. Rails has excellent built-in testing support.

## Why Test?

- **Confidence**: Know your code works
- **Refactoring**: Change code safely
- **Documentation**: Tests show how code should be used
- **Bug Prevention**: Catch issues before production

## Rails Testing Framework

Rails uses **Minitest** by default. Tests live in the `test/` directory:

```
test/
├── controllers/        # Controller tests
├── models/             # Model tests
├── integration/        # Full-stack tests
├── system/             # Browser tests
├── fixtures/           # Test data
├── test_helper.rb      # Test configuration
└── application_system_test_case.rb
```

## Running Tests

```bash
# Run all tests
bin/rails test

# Run specific file
bin/rails test test/models/article_test.rb

# Run specific test by line number
bin/rails test test/models/article_test.rb:10

# Run system tests (browser tests)
bin/rails test:system

# Run with verbose output
bin/rails test -v
```

## Test Structure

Every test follows this pattern:

```ruby
require "test_helper"

class ArticleTest < ActiveSupport::TestCase
  # Setup runs before each test
  setup do
    @article = Article.new(title: "Test", body: "Content")
  end

  test "should be valid with title and body" do
    assert @article.valid?
  end

  test "should require title" do
    @article.title = nil
    assert_not @article.valid?
    assert_includes @article.errors[:title], "can't be blank"
  end
end
```

## Common Assertions

```ruby
# Basic assertions
assert true                          # Passes if truthy
assert_not false                     # Passes if falsy

# Equality
assert_equal expected, actual        # expected == actual
assert_not_equal a, b                # a != b

# Nil checks
assert_nil value                     # value.nil?
assert_not_nil value                 # !value.nil?

# Collections
assert_includes array, item          # array.include?(item)
assert_empty collection              # collection.empty?

# Exceptions
assert_raises(SomeError) { dangerous_code }

# Difference
assert_difference "Article.count", 1 do
  Article.create(title: "New")
end

# No difference
assert_no_difference "Article.count" do
  Article.create(title: "")  # Invalid, not saved
end
```

## Fixtures

Fixtures are sample data for tests:

```yaml
# test/fixtures/articles.yml
published_article:
  title: "Published Article"
  body: "Some content here"
  published: true

draft_article:
  title: "Draft Article"
  body: "Work in progress"
  published: false
```

Use in tests:

```ruby
class ArticleTest < ActiveSupport::TestCase
  test "fixture is loaded" do
    article = articles(:published_article)
    assert_equal "Published Article", article.title
  end
end
```

## Resources

- [Testing Rails Applications](https://guides.rubyonrails.org/testing.html) — Complete guide to testing in Rails

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*