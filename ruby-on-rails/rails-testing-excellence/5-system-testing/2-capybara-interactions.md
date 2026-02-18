---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-capybara-interactions"
---

# Capybara Interactions

Capybara provides an expressive DSL for interacting with web pages.

## Navigation

```ruby
# Visit pages
visit root_url
visit '/articles'

# Go back/forward
go_back
go_forward

# Refresh
refresh
```

## Finding Elements

```ruby
# By text content
find('h1', text: 'Welcome')
find('.article', text: 'My Article')

# By CSS selector
find('#login-form')
find('.nav-link.active')

# By ID
find_by_id('submit-button')

# Find all matching elements
all('.article').each { |el| puts el.text }
```

## Clicking

```ruby
# Click links
click_link 'Sign Up'
click_link 'Articles', match: :first

# Click buttons
click_button 'Submit'
click_on 'Save'  # Clicks link or button

# Click elements
find('.dropdown-toggle').click
```

## Forms

```ruby
# Text fields
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

## Assertions

```ruby
# Content assertions
assert_text 'Welcome back'
assert_no_text 'Error'

# Element assertions
assert_selector 'h1', text: 'Dashboard'
assert_no_selector '.error-message'

# Field assertions
assert_field 'Email', with: 'user@example.com'
assert_checked_field 'Remember me'

# Current path
assert_current_path '/dashboard'
```

## Scoping

```ruby
# Work within a section of the page
within '#sidebar' do
  click_link 'Settings'
end

within '.modal' do
  fill_in 'Name', with: 'New Name'
  click_button 'Save'
end
```

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*