---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-validations"
---

# Testing Model Validations

## Introduction

Validations are the first line of defense for data integrity in a Rails application. Every `validates` declaration in a model should have corresponding tests that prove it accepts good data and rejects bad data. This lesson shows you how to write thorough, maintainable validation tests that catch regressions before they reach production.

## Key Concepts

- **Presence Validation**: Ensures a field is not nil or blank. Test by setting the field to nil and asserting the record is invalid.
- **Uniqueness Validation**: Ensures no two records share the same value for a field. Test by creating a duplicate and checking for the error message.
- **Format Validation**: Ensures a field matches a regular expression. Test with multiple valid and invalid values.
- **Numericality Validation**: Ensures a field is a number with optional constraints (greater than, integer only, etc.). Test boundary values.

## Real World Context

Imagine a team ships a User model where the email uniqueness validation accidentally gets removed during a refactor. Without tests, duplicate emails silently enter the database, causing login failures and data corruption. A simple test like `assert_not duplicate.valid?` would catch this regression immediately in CI. Validation tests are cheap to write and prevent expensive production bugs.

## Deep Dive

### Testing Presence Validations

Presence validations are the most common. Test both the positive case (all fields present) and the negative case (required field missing):

```ruby
class UserTest < ActiveSupport::TestCase
  test 'valid with all required fields' do
    user = User.new(
      name: 'Alice',
      email: 'alice@example.com'
    )
    assert user.valid?
  end

  test 'invalid without email' do
    user = User.new(name: 'Alice')

    assert_not user.valid?
    assert_includes user.errors[:email], "can't be blank"
  end
end
```

Always check the specific error message with `assert_includes user.errors[:field]`. This ensures the right validation triggered, not just any validation.

### Testing Uniqueness Validations

Use fixtures to provide an existing record, then create a duplicate:

```ruby
test 'email must be unique' do
  existing = users(:david)
  duplicate = User.new(
    name: 'Other',
    email: existing.email
  )

  assert_not duplicate.valid?
  assert_includes duplicate.errors[:email], 'has already been taken'
end
```

### Testing Format Validations

Test multiple valid and invalid inputs. Use custom failure messages to identify which value caused a failure:

```ruby
test 'rejects invalid email formats' do
  user = User.new(name: 'Test')

  %w[invalid@ @invalid plaintext].each do |bad_email|
    user.email = bad_email
    assert_not user.valid?, "#{bad_email} should be invalid"
  end
end

test 'accepts valid email formats' do
  user = User.new(name: 'Test')

  %w[user@example.com USER@foo.COM a+b@baz.cn].each do |good_email|
    user.email = good_email
    assert user.valid?, "#{good_email} should be valid"
  end
end
```

### Testing Numericality and Boundary Values

Boundary testing catches off-by-one errors in numericality constraints:

```ruby
test 'price must be positive' do
  product = products(:widget)

  product.price = -1
  assert_not product.valid?

  product.price = 0
  assert_not product.valid?

  product.price = 0.01
  assert product.valid?
end
```

## Common Pitfalls

1. **Only testing the negative case** — If you only test that invalid data fails, you will not catch a bug where the validation is too strict and rejects valid data. Always test both valid and invalid inputs.
2. **Not checking specific error messages** — Using `assert_not user.valid?` alone does not tell you which validation failed. A different validation might be triggering. Always check `errors[:field]` for the expected message.

## Best Practices

1. **Test each validation independently** — Change only one attribute at a time so you know exactly which validation is being exercised. Start from a valid baseline (all attributes set) and remove or modify one.
2. **Test boundary values for numericality** — If the validation says `greater_than: 0`, test with -1, 0, and 0.01. If it says `length: { minimum: 5 }`, test with 4 characters, 5 characters, and 6 characters.

## Summary

- Every validation in the model should have a corresponding test with both positive and negative cases.
- Use `assert_includes record.errors[:field]` to verify the exact error message, not just that the record is invalid.
- Test boundary values for numericality and length validations to catch off-by-one errors.

## Code Examples

**Testing presence and uniqueness validations with specific error message assertions**

```ruby
require 'test_helper'

class UserTest < ActiveSupport::TestCase
  test 'valid with all required fields' do
    user = User.new(name: 'Alice', email: 'alice@example.com')
    assert user.valid?
  end

  test 'invalid without email' do
    user = User.new(name: 'Alice')
    assert_not user.valid?
    assert_includes user.errors[:email], "can't be blank"
  end

  test 'email must be unique' do
    existing = users(:david)
    duplicate = User.new(name: 'Copy', email: existing.email)
    assert_not duplicate.valid?
    assert_includes duplicate.errors[:email], 'has already been taken'
  end
end
```


## Resources

- [Testing Rails Applications - Model Testing](https://guides.rubyonrails.org/testing.html#model-testing) — Official Rails guide section on testing models
- [Active Record Validations](https://guides.rubyonrails.org/active_record_validations.html) — Complete reference for all validation helpers and options

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*