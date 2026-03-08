---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-javascript"
---

# Testing JavaScript Interactions

## Introduction
Modern Rails applications rely heavily on JavaScript for dynamic behavior — modals, live search, Turbo Streams, and Stimulus controllers all produce interactions that cannot be tested without a real browser. System tests with Capybara and headless Chrome execute your JavaScript just as a user's browser would, letting you verify these dynamic features with confidence.

## Key Concepts
- **Automatic Waiting**: Capybara's finders and assertions automatically wait for elements to appear on the page, handling asynchronous JavaScript without manual `sleep` calls.
- **Turbo Streams**: Rails 8's default approach to partial page updates. Turbo replaces or appends HTML fragments without a full page reload.
- **Turbo Frames**: Sections of a page that can be lazily loaded or replaced independently, identified by `<turbo-frame>` tags.
- **JavaScript Dialogs**: Browser-native `confirm()` and `alert()` dialogs that require special Capybara methods to handle.
- **Explicit Wait**: The `wait` option on Capybara assertions that overrides the default wait time for slow operations.

## Real World Context
A common Rails 8 pattern is a Turbo Stream form that appends a new comment to a list without reloading the page. Without system tests, you cannot verify that the stream arrives, the comment appears in the correct position, and the form resets. These are the interactions that break silently in production if left untested.

## Deep Dive

### Capybara's Automatic Waiting
Capybara's most powerful feature is its built-in waiting behavior. When you write `assert_text 'Great post!'`, Capybara does not check once and fail — it retries for up to `Capybara.default_max_wait_time` seconds (2 seconds by default). This handles AJAX responses, Turbo updates, and animation delays without explicit sleeps:

```ruby
test 'modal appears after click' do
  visit articles_url
  click_button 'Delete'

  # Capybara waits for the modal to appear
  within '.modal' do
    assert_text 'Are you sure?'
    click_button 'Confirm'
  end

  # Capybara waits for the modal to disappear
  assert_no_selector '.modal'
end
```

The `within '.modal'` call waits for the modal element to exist before entering the block. The `assert_no_selector` call waits for the element to disappear.

### Testing AJAX and Live Search
For features that update the page asynchronously, Capybara's waiting behavior handles the timing naturally:

```ruby
test 'live search updates results' do
  visit products_url

  fill_in 'Search', with: 'widget'

  within '#search-results' do
    assert_selector '.product', count: 3
    assert_text 'Blue Widget'
  end

  fill_in 'Search', with: ''
  assert_selector '.product', minimum: 10
end
```

After typing "widget", Capybara waits for the search results container to contain exactly 3 product elements. No timing hacks needed.

### Testing Turbo Streams and Frames
Rails 8 applications use Turbo extensively. System tests verify that Turbo Stream responses update the DOM correctly:

```ruby
test 'turbo form submission appends comment' do
  visit new_comment_url

  fill_in 'Comment', with: 'Great post!'
  click_button 'Submit'

  # Turbo Stream appends the comment without page reload
  assert_text 'Great post!'
  assert_selector '#comments .comment', text: 'Great post!'

  # Form should be cleared after submission
  assert_field 'Comment', with: ''
end
```

For Turbo Frames that load content lazily, you can test that the frame content loads:

```ruby
test 'turbo frame loads comments' do
  visit article_url(@article)

  within 'turbo-frame#comments-frame' do
    click_link 'Load More'
    assert_selector '.comment', minimum: 10
  end
end
```

The `within` method scopes actions to a specific Turbo Frame, ensuring your assertions target the correct content.

### Handling JavaScript Dialogs
Browser-native `confirm()` and `alert()` dialogs require special Capybara methods:

```ruby
test 'handles confirm dialog' do
  visit article_url(@article)

  accept_confirm 'Are you sure?' do
    click_button 'Delete'
  end

  assert_current_path articles_path
end

test 'dismisses confirm dialog' do
  visit article_url(@article)

  dismiss_confirm do
    click_button 'Delete'
  end

  # Article should still exist
  assert_text @article.title
end
```

The `accept_confirm` and `dismiss_confirm` methods wrap the action that triggers the dialog. You can optionally assert the dialog's message by passing the expected text.

### Explicit Wait Times
For operations that take longer than the default wait time (like file uploads or external API calls), override the wait on specific assertions:

```ruby
# Wait up to 10 seconds for a slow operation
assert_selector '.loaded', wait: 10

# Or use a block to change the wait time temporarily
using_wait_time(10) do
  assert_text 'Processing complete'
end
```

Use explicit waits sparingly. If you find yourself raising wait times frequently, the underlying feature may need performance optimization.

## Common Pitfalls
1. **Adding `sleep` calls instead of trusting auto-wait** — `sleep 2` makes tests slow and flaky. Capybara's assertions already retry. If they time out, increase `wait` on the specific assertion or investigate why the element is not appearing.
2. **Testing Turbo Frame content without `within`** — Assertions outside a `within` block search the entire page, which can match content from the wrong frame or miss lazy-loaded content entirely.
3. **Forgetting to wrap dialog triggers in accept/dismiss blocks** — Calling `click_button 'Delete'` when it triggers a `confirm()` dialog without wrapping it in `accept_confirm` will hang the test.

## Best Practices
1. **Let Capybara wait for you** — Trust the auto-waiting mechanism. Only use `wait:` overrides for genuinely slow operations, and prefer `assert_selector` over `find` for assertions.
2. **Test Turbo behavior explicitly** — After a Turbo Stream submission, assert that the new content appears AND that the page did not fully reload (check that the form cleared or a counter updated).
3. **Use `assert_dom` for structural assertions in Rails 8** — When you need to verify DOM structure beyond text content, `assert_dom` provides a clean, Rails-idiomatic API.

## Summary
- Capybara automatically waits for elements to appear or disappear, handling AJAX and Turbo updates without `sleep`.
- Use `within '.modal'` and `within` to scope interactions to specific page sections and Turbo Frames.
- Handle browser `confirm()` and `alert()` dialogs with `accept_confirm`, `dismiss_confirm`, and `accept_alert`.
- Override the default wait time with `wait:` on individual assertions for slow operations.
- Test Turbo Stream and Turbo Frame behavior explicitly to verify that partial page updates work correctly.

## Code Examples

**Testing a Turbo Stream form submission — the assertion verifies the comment appears in the DOM and the form resets, all without a full page reload**

```ruby
# Testing a Turbo Stream form that appends new items
class CommentsTest < ApplicationSystemTestCase
  setup do
    @article = articles(:one)
  end

  test 'adding a comment via Turbo Stream' do
    visit article_url(@article)

    fill_in 'Add a comment', with: 'Excellent article!'
    click_button 'Post Comment'

    # Turbo Stream appends without page reload
    assert_selector '#comments .comment', text: 'Excellent article!'
    assert_field 'Add a comment', with: ''  # Form resets
  end
end
```


## Resources

- [Rails System Testing Guide](https://guides.rubyonrails.org/testing.html#system-testing) — Official Rails guide on system tests, including JavaScript interaction patterns
- [Turbo Handbook](https://turbo.hotwired.dev/handbook/introduction) — Official Hotwire Turbo documentation covering Streams, Frames, and Drive

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*