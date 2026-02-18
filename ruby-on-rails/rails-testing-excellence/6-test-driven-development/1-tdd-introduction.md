---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-tdd-introduction"
---

# TDD Introduction

Test-Driven Development (TDD) means writing tests before writing implementation code.

## The Red-Green-Refactor Cycle

1. **Red**: Write a failing test
2. **Green**: Write minimal code to pass the test
3. **Refactor**: Improve the code while keeping tests green

## TDD Example: Building a Calculator

### Step 1: Red - Write Failing Test

```ruby
# test/models/calculator_test.rb
require 'test_helper'

class CalculatorTest < ActiveSupport::TestCase
  test 'adds two numbers' do
    calc = Calculator.new
    assert_equal 5, calc.add(2, 3)
  end
end
```

Run the test - it fails because Calculator doesn't exist.

### Step 2: Green - Minimal Implementation

```ruby
# app/models/calculator.rb
class Calculator
  def add(a, b)
    a + b
  end
end
```

Run the test - it passes.

### Step 3: Refactor (if needed)

In this case, the code is simple enough. Continue with more tests.

## TDD for Features

```ruby
# Start with what you want to happen
test 'user can create a post' do
  user = users(:david)
  sign_in(user)
  
  visit new_post_url
  fill_in 'Title', with: 'TDD Post'
  fill_in 'Body', with: 'Content'
  click_button 'Create'
  
  assert_text 'Post was successfully created'
  assert Post.exists?(title: 'TDD Post')
end
```

Then implement routes, controllers, and views to make it pass.

## Benefits of TDD

- **Design guidance**: Tests help you think about API design
- **Confidence**: Know your code works before shipping
- **Documentation**: Tests describe expected behavior
- **Regression prevention**: Catch bugs early

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*