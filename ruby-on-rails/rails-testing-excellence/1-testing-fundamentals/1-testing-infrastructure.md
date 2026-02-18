---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-infrastructure"
---

# Infrastructure

Rails comes with a powerful testing infrastructure built on Minitest. When you generate a new Rails application, it automatically creates the `test` directory with subdirectories for different types of tests.

## Test Directory Structure

The test directory contains:
- `models/` - Tests for your Active Record models
- `controllers/` - Tests for controller actions
- `integration/` - Tests for interactions between controllers
- `system/` - Full browser tests using Capybara
- `helpers/` - Tests for view helpers
- `mailers/` - Tests for Action Mailer classes
- `fixtures/` - Sample data for your tests
- `test_helper.rb` - Configuration for all tests

## The Test Helper

Every test file requires `test_helper.rb`, which sets up the testing environment:

```ruby
ENV['RAILS_ENV'] ||= 'test'
require_relative '../config/environment'
require 'rails/test_help'

module ActiveSupport
  class TestCase
    # Run tests in parallel
    parallelize(workers: :number_of_processors)
    
    # Setup fixtures in test/fixtures/*.yml
    fixtures :all
  end
end
```

## Running Tests

Rails provides the `bin/rails test` command to run your tests:

```bash
# Run all tests
bin/rails test

# Run specific test file
bin/rails test test/models/user_test.rb

# Run specific test by line number
bin/rails test test/models/user_test.rb:10

# Run tests matching a pattern
bin/rails test test/models/
```

For more details, see the [Testing Rails Applications guide](https://guides.rubyonrails.org/testing.html).

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*