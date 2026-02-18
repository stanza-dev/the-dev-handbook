---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-system-test-setup"
---

# System Test Setup

System tests run in a real browser, testing JavaScript and the full user experience.

## Configuration

Rails configures system tests in `application_system_test_case.rb`:

```ruby
require 'test_helper'

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :chrome, screen_size: [1400, 1400]
end
```

## Headless Browser

For CI environments, use headless Chrome:

```ruby
class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
end
```

## Basic System Test

```ruby
require 'application_system_test_case'

class ArticlesTest < ApplicationSystemTestCase
  test 'viewing the index' do
    visit articles_url
    
    assert_selector 'h1', text: 'Articles'
  end
  
  test 'creating an article' do
    visit articles_url
    click_on 'New Article'
    
    fill_in 'Title', with: 'My New Article'
    fill_in 'Body', with: 'Article content here'
    click_on 'Create Article'
    
    assert_text 'Article was successfully created'
  end
end
```

## Running System Tests

```bash
# Run all system tests
bin/rails test:system

# Run specific system test
bin/rails test test/system/articles_test.rb
```

## Taking Screenshots

Capture screenshots for debugging:

```ruby
test 'debugging with screenshot' do
  visit articles_url
  
  # Take screenshot manually
  take_screenshot
  
  # Screenshots are automatically taken on failure
end
```

See [System Testing](https://guides.rubyonrails.org/testing.html#system-testing) for more details.

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*