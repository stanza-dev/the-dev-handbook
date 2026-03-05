---
source_course: "rails-foundations"
source_lesson: "rails-foundations-testing-introduction"
---

# Introduction to Rails Testing

## Introduction

Testing ensures your application works correctly and continues to work as you make changes. Rails has excellent built-in testing support with Minitest, making it easy to write and run tests from day one.

## Key Concepts

- **Minitest**: Rails' default testing framework, included out of the box with no extra setup.
- **Test directory structure**: Tests mirror the app structure: `test/models/`, `test/controllers/`, `test/integration/`, `test/system/`.
- **Assertions**: Methods like `assert`, `assert_equal`, `assert_not` that verify expected behavior.
- **Fixtures**: YAML files in `test/fixtures/` that define sample data loaded before each test.

## Real World Context

Professional Rails teams require tests for all new code. Tests catch regressions, document expected behavior, and give you confidence to refactor. Without tests, every change is a risk. With tests, you can deploy on Fridays.

## Deep Dive

### Test Directory Structure

```
test/
  controllers/        # Controller/request tests
  models/             # Model tests
  integration/        # Full-stack tests
  system/             # Browser tests
  fixtures/           # Test data (YAML)
  test_helper.rb      # Test configuration
```

### Running Tests

```bash
bin/rails test                              # All tests
bin/rails test test/models/article_test.rb  # Specific file
bin/rails test test/models/article_test.rb:10  # Specific line
bin/rails test:system                       # Browser tests
bin/rails test -v                           # Verbose output
```

### Test Structure

```ruby
require "test_helper"

class ArticleTest < ActiveSupport::TestCase
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

### Common Assertions

```ruby
assert true                          # Passes if truthy
assert_not false                     # Passes if falsy
assert_equal expected, actual        # Equality
assert_nil value                     # Nil check
assert_includes array, item          # Collection membership
assert_raises(SomeError) { code }    # Exception check
assert_difference "Article.count", 1 do
  Article.create(title: "New")       # Count changed by 1
end
```

### Fixtures

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

```ruby
test "fixture is loaded" do
  article = articles(:published_article)
  assert_equal "Published Article", article.title
end
```

## Common Pitfalls

- **Not running tests frequently**: Run tests after every change. Use `bin/rails test` as part of your development workflow.
- **Testing implementation instead of behavior**: Test what your code does, not how it does it. This makes tests resilient to refactoring.
- **Brittle fixtures**: Keep fixtures minimal. Only define the attributes relevant to your tests.

## Best Practices

- Write tests before fixing bugs (test-driven bug fixing) to ensure the bug stays fixed.
- Use `setup` blocks for shared test setup instead of repeating code.
- Run the full test suite before pushing code.

## Summary

- Rails uses Minitest by default with tests in the `test/` directory.
- Run tests with `bin/rails test` and target specific files or line numbers.
- Assertions verify expected outcomes: `assert`, `assert_equal`, `assert_difference`.
- Fixtures provide sample test data defined in YAML files.
- Every test follows: setup, exercise, verify (arrange, act, assert).

## Code Examples

**A model test verifying validations using Minitest assertions. Each test method describes the expected behavior and uses assertions to verify it.**

```ruby
require "test_helper"

class ArticleTest < ActiveSupport::TestCase
  test "should be valid with required attributes" do
    article = Article.new(title: "Test", body: "Content")
    assert article.valid?
  end

  test "should require title" do
    article = Article.new(body: "Content")
    assert_not article.valid?
    assert_includes article.errors[:title], "can't be blank"
  end
end
```


## Resources

- [Testing Rails Applications](https://guides.rubyonrails.org/testing.html) — Complete guide to testing in Rails

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*