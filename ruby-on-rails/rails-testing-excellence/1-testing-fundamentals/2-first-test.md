---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-writing-first-test"
---

# Writing First Test

Let's write your first Rails test. Tests in Rails use Minitest, which provides simple and straightforward assertion methods.

## Test Structure

A basic test file follows this pattern:

```ruby
require 'test_helper'

class ArticleTest < ActiveSupport::TestCase
  test 'the truth' do
    assert true
  end
end
```

The `test` method creates a test case with a descriptive name. Inside the block, you use assertions to verify expected behavior.

## Common Assertions

Minitest provides many assertion methods:

```ruby
# Basic assertions
assert expression              # Passes if expression is truthy
assert_not expression          # Passes if expression is falsy

# Equality assertions
assert_equal expected, actual  # Passes if expected == actual
assert_not_equal a, b          # Passes if a != b

# Nil assertions
assert_nil object              # Passes if object is nil
assert_not_nil object          # Passes if object is not nil

# Collection assertions
assert_empty collection        # Passes if collection is empty
assert_includes collection, obj # Passes if collection includes obj

# Exception assertions
assert_raises(ErrorClass) { block }  # Passes if block raises ErrorClass
```

## Setup and Teardown

Use `setup` and `teardown` for test preparation and cleanup:

```ruby
class ArticleTest < ActiveSupport::TestCase
  def setup
    @article = Article.new(title: 'Test', body: 'Content')
  end
  
  def teardown
    @article = nil
  end
  
  test 'article has title' do
    assert_equal 'Test', @article.title
  end
end
```

Learn more at [Testing Rails Applications](https://guides.rubyonrails.org/testing.html#available-assertions).

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*