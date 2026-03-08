---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-tdd-introduction"
---

# TDD Introduction

## Introduction
Test-Driven Development (TDD) flips the traditional coding workflow: you write the test first, watch it fail, then write just enough code to make it pass. This seemingly counterintuitive approach produces cleaner APIs, fewer bugs, and code that is inherently testable. Rails embraces TDD by generating test files alongside every model, controller, and mailer you scaffold.

## Key Concepts
- **Red-Green-Refactor**: The three-phase TDD cycle. Red means write a failing test, Green means write minimal code to pass it, and Refactor means improve the code while keeping tests green.
- **Failing Test First**: The core TDD discipline — never write production code without a failing test that demands it.
- **Minimal Implementation**: In the Green phase, write the simplest code that makes the test pass. Resist the urge to anticipate future requirements.
- **Refactoring Under Green**: Once the test passes, improve code structure (extract methods, rename variables, remove duplication) while running tests continuously to ensure nothing breaks.

## Real World Context
Without TDD, developers often write code first and tests later — if at all. This leads to tests that merely confirm existing behavior rather than driving design decisions. TDD forces you to think about the API from the caller's perspective before you write a single line of implementation. Teams practicing TDD report fewer production bugs, faster debugging cycles, and codebases that are easier to modify because every behavior is backed by a test.

## Deep Dive
The TDD cycle has three distinct phases, each with a specific purpose.

### Phase 1: Red — Write a Failing Test
Start by describing what you want the code to do. In Rails 8, generators create test files automatically when you scaffold resources:

```bash
bin/rails generate model Calculator
# Creates app/models/calculator.rb AND test/models/calculator_test.rb
```

Open the test file and write the behavior you want:

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

Run the test with `bin/rails test test/models/calculator_test.rb`. It fails because `Calculator` has no `add` method. This is the Red phase — the failure confirms your test is actually testing something.

### Phase 2: Green — Minimal Implementation
Write the simplest code that makes the test pass:

```ruby
# app/models/calculator.rb
class Calculator
  def add(a, b)
    a + b
  end
end
```

Run the test again. It passes. You are now in the Green phase. Resist the urge to add `subtract`, `multiply`, or error handling — those behaviors do not have tests yet.

### Phase 3: Refactor
With a passing test as your safety net, improve the code. In this simple example there is nothing to refactor, but in real applications this phase might involve extracting a class, renaming a method, or eliminating duplication.

The key rule: run `bin/rails test` after every change. If a test goes red during refactoring, you introduced a regression — undo the last change and try again.

### A Realistic TDD Example
Let us build a `Post` model's publishing behavior using TDD:

```ruby
# test/models/post_test.rb
class PostTest < ActiveSupport::TestCase
  test 'publish! sets published_at to current time' do
    post = Post.new(title: 'TDD Guide')
    assert_nil post.published_at

    travel_to Time.zone.local(2026, 3, 8, 12, 0, 0) do
      post.publish!
      assert_equal Time.zone.local(2026, 3, 8, 12, 0, 0), post.published_at
    end
  end

  test 'published? returns true for published posts' do
    post = Post.new(published_at: 1.hour.ago)
    assert post.published?
  end

  test 'published? returns false for draft posts' do
    post = Post.new(published_at: nil)
    assert_not post.published?
  end
end
```

All three tests fail (Red). Now implement the minimum code:

```ruby
# app/models/post.rb
class Post < ApplicationRecord
  def publish!
    update!(published_at: Time.current)
  end

  def published?
    published_at.present?
  end
end
```

All three tests pass (Green). The `travel_to` helper freezes time during the test, making the assertion deterministic.

## Common Pitfalls
1. **Writing too much code in the Green phase** — TDD discipline means writing only the code required by the current failing test. Adding methods that no test demands leads to untested code and bloated APIs.
2. **Skipping the Red phase** — If your test passes on the first run, either the behavior already exists (and the test is redundant) or the test is not actually checking what you think. Always see the failure first.
3. **Refactoring while Red** — Never change production code structure when tests are failing. Get back to Green first, then refactor.

## Best Practices
1. **Run tests after every small change** — Use `bin/rails test` frequently. The faster your feedback loop, the easier it is to pinpoint what broke.
2. **Write one test at a time** — Write a single test, make it pass, then write the next one. Batching multiple failing tests makes debugging harder.
3. **Use Rails generators to start with test files** — `bin/rails generate model Foo` creates both the model and its test file. Rails 8 generators set up the test structure for you.

## Summary
- TDD follows the Red-Green-Refactor cycle: failing test, minimal implementation, then code improvement.
- Always see the test fail before writing implementation code — this validates the test itself.
- Write the simplest code that passes the test. Do not anticipate future requirements.
- Refactor only when tests are green, running `bin/rails test` after every change.
- Rails 8 generators create test files automatically, making it easy to start with TDD from the beginning.

## Code Examples

**The complete Red-Green-Refactor cycle — write the test first, implement the minimum to pass, then refactor with confidence**

```ruby
# Step 1: RED — Write a failing test
class CalculatorTest < ActiveSupport::TestCase
  test 'adds two numbers' do
    calc = Calculator.new
    assert_equal 5, calc.add(2, 3)
  end
end

# Step 2: GREEN — Minimal implementation
class Calculator
  def add(a, b)
    a + b
  end
end

# Step 3: REFACTOR — Improve if needed (nothing to change here)
# Run: bin/rails test — all green!
```


## Resources

- [Rails Testing Guide](https://guides.rubyonrails.org/testing.html) — Official Rails guide on testing, covering all test types and the testing infrastructure

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*