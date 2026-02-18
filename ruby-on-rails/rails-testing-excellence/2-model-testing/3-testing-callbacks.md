---
source_course: "rails-testing-excellence"
source_lesson: "rails-testing-testing-callbacks"
---

# Testing Callbacks

Callbacks and custom methods contain business logic that needs thorough testing.

## Testing before_save Callbacks

```ruby
class User < ApplicationRecord
  before_save :normalize_email
  
  private
  
  def normalize_email
    self.email = email.downcase.strip
  end
end

class UserTest < ActiveSupport::TestCase
  test 'email is normalized before save' do
    user = User.create!(
      name: 'Test',
      email: '  USER@EXAMPLE.COM  '
    )
    
    assert_equal 'user@example.com', user.email
  end
end
```

## Testing after_create Callbacks

```ruby
class Order < ApplicationRecord
  after_create :send_confirmation_email
  
  private
  
  def send_confirmation_email
    OrderMailer.confirmation(self).deliver_later
  end
end

class OrderTest < ActiveSupport::TestCase
  test 'sends confirmation email after creation' do
    assert_enqueued_emails 1 do
      Order.create!(customer: 'John', total: 100)
    end
  end
end
```

## Testing Custom Methods

```ruby
class User < ApplicationRecord
  def full_name
    [first_name, last_name].compact.join(' ')
  end
  
  def adult?
    age.present? && age >= 18
  end
end

class UserTest < ActiveSupport::TestCase
  test 'full_name combines first and last name' do
    user = User.new(first_name: 'John', last_name: 'Doe')
    assert_equal 'John Doe', user.full_name
  end
  
  test 'full_name handles missing last name' do
    user = User.new(first_name: 'John')
    assert_equal 'John', user.full_name
  end
  
  test 'adult? returns true for age 18+' do
    assert User.new(age: 18).adult?
    assert User.new(age: 25).adult?
  end
  
  test 'adult? returns false for under 18' do
    assert_not User.new(age: 17).adult?
    assert_not User.new(age: nil).adult?
  end
end
```

## Testing Scopes

```ruby
class Article < ApplicationRecord
  scope :published, -> { where(published: true) }
  scope :recent, -> { where('created_at > ?', 1.week.ago) }
end

class ArticleTest < ActiveSupport::TestCase
  test 'published scope returns only published articles' do
    published = articles(:published_one, :published_two)
    draft = articles(:draft)
    
    result = Article.published
    
    assert_includes result, published.first
    assert_not_includes result, draft
  end
end
```

---

> 📘 *This lesson is part of the [Rails Testing Excellence](https://stanza.dev/courses/rails-testing-excellence) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*