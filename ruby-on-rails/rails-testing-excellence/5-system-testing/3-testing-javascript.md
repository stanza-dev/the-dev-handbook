---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-javascript"
---

# Testing Javascript

System tests can interact with JavaScript-powered features.

## Waiting for JavaScript

Capybara automatically waits for elements:

```ruby
test 'modal appears after click' do
  visit articles_url
  
  click_button 'Delete'
  
  # Capybara waits for modal to appear
  within '.modal' do
    assert_text 'Are you sure?'
    click_button 'Confirm'
  end
  
  # Waits for modal to disappear
  assert_no_selector '.modal'
end
```

## AJAX Interactions

```ruby
test 'live search updates results' do
  visit products_url
  
  # Type in search box (triggers AJAX)
  fill_in 'Search', with: 'widget'
  
  # Capybara waits for results to update
  within '#search-results' do
    assert_selector '.product', count: 3
    assert_text 'Blue Widget'
  end
  
  # Clear search
  fill_in 'Search', with: ''
  assert_selector '.product', minimum: 10
end
```

## Testing Turbo/Hotwire

```ruby
test 'turbo form submission' do
  visit new_comment_url
  
  fill_in 'Comment', with: 'Great post!'
  click_button 'Submit'
  
  # Page updates via Turbo Stream
  assert_text 'Great post!'
  assert_selector '#comments .comment', text: 'Great post!'
  
  # Form should be cleared
  assert_field 'Comment', with: ''
end

test 'turbo frame updates' do
  visit article_url(@article)
  
  within_frame 'comments-frame' do
    click_link 'Load More'
    assert_selector '.comment', minimum: 10
  end
end
```

## Handling JavaScript Dialogs

```ruby
test 'handles confirm dialog' do
  visit article_url(@article)
  
  accept_confirm 'Are you sure?' do
    click_button 'Delete'
  end
  
  assert_current_path articles_path
end

test 'handles alert' do
  visit some_url
  
  accept_alert do
    click_button 'Show Alert'
  end
end
```

## Explicit Waiting

```ruby
# Wait for specific condition
assert_selector '.loaded', wait: 10

# Using Capybara's using_wait_time
using_wait_time(10) do
  assert_text 'Loaded!'
end
```

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*