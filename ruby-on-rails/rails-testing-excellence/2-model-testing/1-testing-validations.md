---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-validations"
---

# Testing Validations

Testing validations ensures your data integrity rules work correctly. Every validation should have corresponding tests.

## Testing Presence Validations

```ruby
class UserTest < ActiveSupport::TestCase
  test 'email is required' do
    user = User.new(name: 'Test')
    
    assert_not user.valid?
    assert_includes user.errors[:email], "can't be blank"
  end
  
  test 'valid with all required fields' do
    user = User.new(
      name: 'Test',
      email: 'test@example.com'
    )
    assert user.valid?
  end
end
```

## Testing Uniqueness Validations

```ruby
class UserTest < ActiveSupport::TestCase
  test 'email must be unique' do
    existing = users(:david)
    duplicate = User.new(
      name: 'Other',
      email: existing.email
    )
    
    assert_not duplicate.valid?
    assert_includes duplicate.errors[:email], 'has already been taken'
  end
end
```

## Testing Format Validations

```ruby
class UserTest < ActiveSupport::TestCase
  test 'email format validation' do
    user = User.new(name: 'Test')
    
    # Test invalid formats
    %w[invalid@ @invalid invalid].each do |invalid_email|
      user.email = invalid_email
      assert_not user.valid?, "#{invalid_email} should be invalid"
    end
    
    # Test valid formats
    %w[user@example.com USER@foo.COM a+b@baz.cn].each do |valid_email|
      user.email = valid_email
      assert user.valid?, "#{valid_email} should be valid"
    end
  end
end
```

## Testing Numericality Validations

```ruby
class ProductTest < ActiveSupport::TestCase
  test 'quantity must be positive integer' do
    product = products(:widget)
    
    product.quantity = -1
    assert_not product.valid?
    
    product.quantity = 0
    assert_not product.valid?
    
    product.quantity = 1.5
    assert_not product.valid?
    
    product.quantity = 1
    assert product.valid?
  end
end
```

See [Active Record Validations](https://guides.rubyonrails.org/active_record_validations.html) for all validation types.

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*