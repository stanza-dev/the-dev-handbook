---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-understanding-fixtures"
---

# Understanding Fixtures

Fixtures provide sample data for your tests. They are YAML files that define database records loaded before each test.

## Fixture Files

Fixtures are stored in `test/fixtures/` with names matching your model tables:

```yaml
# test/fixtures/users.yml
david:
  name: David
  email: david@example.com
  admin: true

steve:
  name: Steve
  email: steve@example.com
  admin: false
```

Each fixture has a name (like `david`) that becomes a method to access it in tests.

## Accessing Fixtures

In your tests, access fixtures as Active Record objects:

```ruby
class UserTest < ActiveSupport::TestCase
  test 'david is an admin' do
    # Access the fixture
    david = users(:david)
    
    assert david.admin?
    assert_equal 'David', david.name
  end
  
  test 'can get multiple fixtures' do
    # Get multiple fixtures at once
    admins = users(:david, :steve)
    assert_equal 2, admins.size
  end
end
```

## Fixture Associations

Use labels to create associations between fixtures:

```yaml
# test/fixtures/posts.yml
first_post:
  title: Welcome
  body: Hello World
  user: david  # References the david fixture

# test/fixtures/comments.yml
first_comment:
  body: Great post!
  post: first_post
  user: steve
```

## ERB in Fixtures

You can use ERB for dynamic values:

```yaml
# test/fixtures/users.yml
<% 10.times do |n| %>
user_<%= n %>:
  name: User <%= n %>
  email: user<%= n %>@example.com
<% end %>
```

See [Fixtures documentation](https://guides.rubyonrails.org/testing.html#the-low-down-on-fixtures) for more details.

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*