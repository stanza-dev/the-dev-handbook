---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-capybara-interactions"
---

# Capybara Interactions

## Introduction
Capybara provides a rich, expressive DSL for interacting with web pages in your system tests. Rather than manipulating raw HTML or sending HTTP requests, you use human-readable methods like `click_on`, `fill_in`, and `assert_text`. Mastering this DSL is the key to writing system tests that are both powerful and maintainable.

## Key Concepts
- **Navigation Methods**: Methods like `visit`, `go_back`, and `refresh` that control which page the browser displays.
- **Finder Methods**: Methods like `find`, `find_by_id`, and `all` that locate elements on the page using CSS selectors or text content.
- **Interaction Methods**: Methods like `click_on`, `fill_in`, `select`, `check`, and `attach_file` that simulate user actions on form elements.
- **Assertion Methods**: Methods like `assert_text`, `assert_selector`, and `assert_current_path` that verify the page state.
- **Scoping**: The `within` block that restricts subsequent actions and assertions to a specific section of the page.

## Real World Context
In a production application, a single page may contain dozens of forms, buttons, and interactive elements. Without Capybara's scoping and precise selectors, your tests would be brittle and ambiguous — clicking the wrong "Submit" button or asserting text that appears in the wrong section. Capybara's DSL lets you write tests that mirror how a QA engineer would describe a test scenario: "within the cart sidebar, click Remove, then verify the total updates."

## Deep Dive

### Navigation
Every system test begins by visiting a page. Capybara's navigation methods control the browser's location:

```ruby
# Visit a URL
visit root_url
visit '/articles'

# Browser history
go_back
go_forward

# Reload the page
refresh
```

Always use Rails URL helpers (like `root_url`, `articles_url`) rather than hardcoded paths. This keeps your tests resilient to route changes.

### Finding Elements
Capybara can locate elements by CSS selector, text content, or ID:

```ruby
# By CSS selector and text
find('h1', text: 'Welcome')
find('.article', text: 'My Article')

# By CSS selector alone
find('#login-form')
find('.nav-link.active')

# By ID
find_by_id('submit-button')

# All matching elements
all('.article').each { |el| puts el.text }
```

The `find` method waits automatically for the element to appear (up to Capybara's default wait time), which handles AJAX-loaded content gracefully.

### Clicking
Capybara distinguishes between links and buttons, but `click_on` handles both:

```ruby
# Click links by text
click_link 'Sign Up'
click_link 'Articles', match: :first  # When multiple links match

# Click buttons by text
click_button 'Submit'

# Click either links or buttons
click_on 'Save'

# Click arbitrary elements
find('.dropdown-toggle').click
```

The `match: :first` option resolves ambiguity when multiple elements share the same text. Use it sparingly — prefer more specific selectors.

### Form Interactions
Capybara interacts with every standard form element:

```ruby
# Text fields — by label text or name attribute
fill_in 'Email', with: 'user@example.com'
fill_in 'user_email', with: 'user@example.com'

# Select dropdowns
select 'California', from: 'State'

# Checkboxes
check 'Remember me'
uncheck 'Subscribe to newsletter'

# Radio buttons
choose 'Credit Card'

# File uploads
attach_file 'Avatar', Rails.root.join('test/fixtures/files/avatar.jpg')
```

Capybara finds fields by their associated `<label>` text, which encourages accessible HTML. If your form fields lack labels, Capybara forces you to use less readable selectors — a signal to fix your markup.

### Assertions
Capybara provides assertion methods that wait for the condition to become true:

```ruby
# Content
assert_text 'Welcome back'
assert_no_text 'Error'

# Elements
assert_selector 'h1', text: 'Dashboard'
assert_no_selector '.error-message'

# Form state
assert_field 'Email', with: 'user@example.com'
assert_checked_field 'Remember me'

# URL
assert_current_path '/dashboard'
```

The `assert_no_*` methods wait for the element to disappear, which is critical for testing Turbo removals and modal closings.

### Scoping with `within`
The `within` block restricts all actions to a specific part of the page:

```ruby
within '#sidebar' do
  click_link 'Settings'
end

within '.modal' do
  fill_in 'Name', with: 'New Name'
  click_button 'Save'
end
```

Scoping prevents your test from accidentally interacting with the wrong element when the page has duplicate buttons or links.

## Common Pitfalls
1. **Using CSS selectors instead of labels** — `fill_in '#user_email'` works but is brittle. Prefer `fill_in 'Email', with: ...` which uses the label, mirrors user behavior, and enforces accessibility.
2. **Forgetting that Capybara auto-waits** — Adding manual `sleep` calls before assertions is unnecessary and slows your tests. Capybara's finders and assertions already wait up to `Capybara.default_max_wait_time` seconds.
3. **Not scoping interactions** — When a page has two "Delete" buttons, clicking without `within` may hit the wrong one. Always scope to the relevant container.

## Best Practices
1. **Use label text for form interactions** — `fill_in 'Email'` reads like a user instruction and ensures your forms are accessible.
2. **Prefer `click_on` over `click_link`/`click_button`** — Unless you specifically need to assert the element type, `click_on` is more resilient to markup changes.
3. **Scope with `within` for complex pages** — Any page with repeated UI elements (tables, cards, modals) should use `within` to target the correct section.

## Summary
- Capybara provides navigation (`visit`), finder (`find`), interaction (`click_on`, `fill_in`), and assertion (`assert_text`, `assert_selector`) methods.
- Use label text for form fields to mirror real user behavior and enforce accessibility.
- The `within` block scopes actions to a specific part of the page, preventing ambiguous interactions.
- Capybara auto-waits for elements to appear or disappear, so avoid manual `sleep` calls.
- Use `match: :first` or `within` when multiple elements share the same text.

## Code Examples

**A system test combining scoped interactions with within blocks, form filling, and assertions — notice how each action targets a specific page region**

```ruby
# A complete system test demonstrating navigation, forms, and assertions
class ProductsTest < ApplicationSystemTestCase
  test 'adding a product to the cart' do
    visit products_url

    within '.product-card', text: 'Ruby Cookbook' do
      click_on 'Add to Cart'
    end

    within '#cart-sidebar' do
      assert_text 'Ruby Cookbook'
      assert_selector '.cart-item', count: 1
      fill_in 'Quantity', with: '2'
      click_on 'Update'
    end

    assert_text 'Cart updated'
  end
end
```


## Resources

- [Capybara README](https://github.com/teamcapybara/capybara) — Capybara's official repository with full DSL documentation and usage examples

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*