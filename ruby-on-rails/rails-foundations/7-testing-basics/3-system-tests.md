---
source_course: "rails-foundations"
source_lesson: "rails-foundations-system-tests"
---

# System Tests and Integration Testing

## Introduction

System tests drive a real browser to test your application from the user's perspective. They click buttons, fill in forms, and verify that the right content appears. While slower than unit tests, system tests catch integration bugs no other test type can find.

## Key Concepts

- **System test**: A test that drives a real or headless browser via Capybara to interact with your full application stack.
- **Capybara**: A Ruby library providing a DSL for web page interaction (`visit`, `fill_in`, `click_on`).
- **`driven_by`**: Configures which browser driver to use (`:selenium_headless`, `:rack_test`).
- **Headless browser**: A browser without a visible window, used in CI environments.

## Real World Context

System tests verify the full stack: routes, controllers, views, JavaScript, and database all working together. A unit test confirms a model validates correctly, but only a system test confirms a user can actually fill in the registration form and see their dashboard.

## Deep Dive

### Writing a System Test

```ruby
require "application_system_test_case"

class ArticlesTest < ApplicationSystemTestCase
  test "creating an article" do
    visit new_article_path

    fill_in "Title", with: "My New Article"
    fill_in "Body", with: "This is the article body."
    click_on "Create Article"

    assert_text "Article was successfully created"
    assert_selector "h1", text: "My New Article"
  end
end
```

Capybara methods (`visit`, `fill_in`, `click_on`) interact with the page as a real user would. Assertion methods verify what appears on screen.

### The Base Test Case

```ruby
# test/application_system_test_case.rb
require "test_helper"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
end
```

The `driven_by` method configures the browser. `:headless_chrome` runs Chrome without a visible window.

### Common Capybara Actions

```ruby
visit articles_path
fill_in "Email", with: "user@example.com"
select "Published", from: "Status"
check "Terms of Service"
click_on "Submit"

assert_text "Welcome back"
assert_selector ".alert-success"
assert_no_text "Error"
assert_current_path dashboard_path
```

Capybara automatically waits for elements to appear, making tests resilient to asynchronous behavior.

## Common Pitfalls

- **Testing implementation details**: Test user behavior, not internals.
- **Not using headless mode in CI**: Running with a visible browser on CI will fail.
- **Slow test suites**: Focus system tests on critical user flows.

## Best Practices

- Write system tests for critical journeys: signup, login, core workflows.
- Use `assert_text` and `assert_selector` rather than checking raw HTML.
- Run system tests separately: `bin/rails test:system`.

## Summary

- System tests use Capybara to drive a real browser and test the full stack.
- Use `visit`, `fill_in`, `click_on` to interact with pages as a user would.
- `assert_text` and `assert_selector` verify visible page content.
- Configure the browser with `driven_by` — use `:headless_chrome` for CI.
- Focus system tests on critical user flows to keep the suite fast.

## Code Examples

**A system test simulating a user creating an article — Capybara drives the browser to fill in fields and verify the result.**

```ruby
require "application_system_test_case"

class ArticlesTest < ApplicationSystemTestCase
  test "user creates an article" do
    visit new_article_path
    fill_in "Title", with: "Rails Testing Guide"
    fill_in "Body", with: "System tests are powerful."
    click_on "Create Article"

    assert_text "Article was successfully created"
    assert_selector "h1", text: "Rails Testing Guide"
  end
end
```


## Resources

- [Testing Rails - System Tests](https://guides.rubyonrails.org/testing.html#system-testing) — Official guide to writing system tests with Capybara

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*