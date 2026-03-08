---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-writing-first-test"
---

# Writing Your First Test

## Introduction

Now that you understand the testing infrastructure, it is time to write your first test. Rails tests use Minitest, which provides a simple, readable syntax for defining test cases and verifying behavior with assertions. By the end of this lesson you will know how to create a test file, write test methods, and use setup hooks to keep your tests clean.

## Key Concepts

- **Test Case Class**: A Ruby class that inherits from `ActiveSupport::TestCase` (for models) or a more specific base class. Each public method defined with the `test` macro becomes a runnable test.
- **Assertion**: A method call that verifies an expected condition. If the condition is false, the test fails with a descriptive message.
- **Setup and Teardown**: Lifecycle hooks that run before and after each test method, used to prepare test data and clean up resources.
- **Test Naming**: The string passed to the `test` macro becomes the test name. Good names describe the expected behavior, not the implementation.

## Real World Context

Every pull request on a professional Rails project is expected to include tests. Reviewers check that new features have corresponding test cases and that bug fixes include a regression test proving the bug is resolved. Writing your first test is the gateway to this workflow — once you can express expected behavior as assertions, you can contribute confidently to any Rails codebase.

## Deep Dive

A basic test file follows this structure:

```ruby
require 'test_helper'

class ArticleTest < ActiveSupport::TestCase
  test 'the truth' do
    assert true
  end
end
```

The `test` method is a Rails convenience that creates a method named `test_the_truth` under the hood. Inside the block you write one or more assertions. If every assertion passes, the test passes.

Here is a more realistic example that tests an Article model:

```ruby
require 'test_helper'

class ArticleTest < ActiveSupport::TestCase
  def setup
    @article = Article.new(
      title: 'Testing in Rails',
      body: 'A comprehensive guide to writing tests.'
    )
  end

  test 'valid with title and body' do
    assert @article.valid?
  end

  test 'invalid without title' do
    @article.title = nil
    assert_not @article.valid?
  end

  test 'title is accessible' do
    assert_equal 'Testing in Rails', @article.title
  end
end
```

The `setup` method runs before every test, giving each test a fresh `@article` instance. This prevents tests from affecting each other through shared state.

Common assertions you will use constantly:

```ruby
# Boolean assertions
assert expression                # passes if expression is truthy
assert_not expression            # passes if expression is falsy

# Equality
assert_equal expected, actual    # passes if expected == actual
assert_not_equal a, b            # passes if a != b

# Nil checks
assert_nil object                # passes if object is nil
assert_not_nil object            # passes if object is not nil

# Collections
assert_empty collection          # passes if collection is empty
assert_includes collection, obj  # passes if collection includes obj

# Exceptions
assert_raises(ActiveRecord::RecordInvalid) { block }  # passes if block raises the error
```

Notice that `assert_equal` takes the expected value first and the actual value second. Getting this order right matters because failure messages use it: `Expected: X, Actual: Y`.

## Common Pitfalls

1. **Reversing assert_equal arguments** — Writing `assert_equal actual, expected` produces confusing failure messages. Always put the expected value first: `assert_equal expected, actual`.
2. **Sharing mutable state between tests** — If you assign instance variables outside of `setup`, one test can modify them and break another. Always initialize test data in `setup` so each test starts fresh.

## Best Practices

1. **One assertion concept per test** — Each test should verify one behavior. If a test has five unrelated assertions, split it into five tests with descriptive names.
2. **Name tests as behavior descriptions** — Use names like `test 'rejects negative prices'` instead of `test 'test_price_validation'`. Good names serve as living documentation.

## Summary

- Test files inherit from `ActiveSupport::TestCase` and use the `test` macro to define individual test cases.
- Assertions like `assert`, `assert_equal`, and `assert_not` verify expected behavior and produce clear failure messages.
- The `setup` method initializes fresh test data before each test, preventing cross-test contamination.

## Code Examples

**A complete model test with setup, positive and negative validation tests, and equality assertion**

```ruby
require 'test_helper'

class ArticleTest < ActiveSupport::TestCase
  def setup
    @article = Article.new(
      title: 'Testing in Rails',
      body: 'A comprehensive guide.'
    )
  end

  test 'valid with title and body' do
    assert @article.valid?
  end

  test 'invalid without title' do
    @article.title = nil
    assert_not @article.valid?
  end

  test 'title is accessible' do
    assert_equal 'Testing in Rails', @article.title
  end
end
```


## Resources

- [Testing Rails Applications - Your First Test](https://guides.rubyonrails.org/testing.html#your-first-failing-test) — The official Rails guide section on writing your first test
- [Minitest Assertions Reference](https://guides.rubyonrails.org/testing.html#available-assertions) — Complete list of available Minitest assertions in Rails

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*