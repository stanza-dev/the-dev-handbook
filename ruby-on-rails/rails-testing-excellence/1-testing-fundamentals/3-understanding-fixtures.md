---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-understanding-fixtures"
---

# Understanding Fixtures

## Introduction
Fixtures provide a convenient way to set up sample data for your tests. Instead of manually creating records in every test method, fixtures let you define reusable test data in YAML files that Rails loads automatically before each test runs.

## Key Concepts
- **Fixture**: A YAML file defining sample database records, stored in `test/fixtures/`.
- **Fixture label**: A unique name (like `david`) that becomes a method to access the record in tests.
- **Transactional fixtures**: Rails loads fixtures inside a transaction that rolls back after each test, keeping tests isolated.

## Real World Context
Every Rails application needs consistent test data. Without fixtures, you'd write repetitive setup code in every test file. Fixtures give you a shared, declarative data set that makes tests cleaner and faster — the records exist before your test even runs.

## Deep Dive

Fixtures are stored in `test/fixtures/` as YAML files named after your database tables.

Here is a basic user fixture file:

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

Each top-level key (`david`, `steve`) is a fixture label. Rails creates a real database record for each one.

Access fixtures in your tests using the table name as a method and the label as a symbol argument:

```ruby
class UserTest < ActiveSupport::TestCase
  test 'david is an admin' do
    david = users(:david)
    assert david.admin?
    assert_equal 'David', david.name
  end
end
```

The `users(:david)` call returns a full Active Record object loaded from the database.

### Fixture Associations

Reference other fixtures by their label to set up associations:

```yaml
# test/fixtures/posts.yml
first_post:
  title: Welcome
  body: Hello World
  user: david

# test/fixtures/comments.yml
first_comment:
  body: Great post!
  post: first_post
  user: steve
```

Rails resolves `user: david` to the `david` fixture's ID, creating a proper foreign key relationship.

### ERB in Fixtures

Use ERB for dynamic or generated values:

```yaml
# test/fixtures/users.yml
<% 10.times do |n| %>
user_<%= n %>:
  name: User <%= n %>
  email: user<%= n %>@example.com
<% end %>
```

This generates 10 user fixtures without repetition.

## Common Pitfalls
1. **Fixture data bypasses model validations** — Fixtures are inserted directly into the database, so invalid data won't raise errors at load time. Always ensure your fixture data is realistic and valid.
2. **Overusing fixtures for complex scenarios** — Fixtures work best for baseline data. For test-specific setups with many variations, create records inline with `User.create!` instead of adding dozens of fixture entries.

## Best Practices
1. **Keep fixtures minimal** — Define only the fixtures you actually reference in tests. A handful of well-named fixtures is better than dozens of unused ones.
2. **Use meaningful labels** — Name fixtures after their role (`admin_user`, `published_post`) rather than generic names (`user1`, `post2`) so tests read clearly.

## Summary
- Fixtures are YAML files in `test/fixtures/` that define reusable test data.
- Access them with `model_name(:label)` to get Active Record objects.
- Use labels to set up associations between fixture records.
- ERB is supported for generating dynamic fixture data.
- Fixtures bypass model validations — keep your fixture data realistic.

## Code Examples

**Accessing fixtures in tests — use the table name method with the fixture label as a symbol argument**

```ruby
# Access fixtures in tests
class PostTest < ActiveSupport::TestCase
  test 'post belongs to user' do
    post = posts(:first_post)
    assert_equal users(:david), post.user
  end

  test 'can get multiple fixtures' do
    all_posts = posts(:first_post, :second_post)
    assert_equal 2, all_posts.size
  end
end
```


## Resources

- [The Low-Down on Fixtures](https://guides.rubyonrails.org/testing.html#the-low-down-on-fixtures) — Official Rails guide on fixtures — setup, associations, ERB, and best practices

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*