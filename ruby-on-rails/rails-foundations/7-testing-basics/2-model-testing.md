---
source_course: "rails-foundations"
source_lesson: "rails-foundations-model-testing"
---

# Testing Models

Model tests verify your business logic, validations, associations, and methods work correctly.

## Generating Model Tests

```bash
bin/rails generate model Article title:string body:text
# Creates: test/models/article_test.rb
```

## Testing Validations

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  validates :title, presence: true, length: { minimum: 5 }
  validates :body, presence: true
end
```

```ruby
# test/models/article_test.rb
require "test_helper"

class ArticleTest < ActiveSupport::TestCase
  test "valid with all attributes" do
    article = Article.new(title: "Hello World", body: "Content")
    assert article.valid?
  end

  test "invalid without title" do
    article = Article.new(body: "Content")
    assert_not article.valid?
    assert_includes article.errors[:title], "can't be blank"
  end

  test "invalid with short title" do
    article = Article.new(title: "Hi", body: "Content")
    assert_not article.valid?
    assert_includes article.errors[:title], "is too short (minimum is 5 characters)"
  end

  test "invalid without body" do
    article = Article.new(title: "Hello World")
    assert_not article.valid?
  end
end
```

## Testing Associations

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  belongs_to :author, class_name: "User"
  has_many :comments, dependent: :destroy
end
```

```ruby
# test/models/article_test.rb
class ArticleTest < ActiveSupport::TestCase
  test "belongs to author" do
    article = articles(:published_article)
    assert_respond_to article, :author
    assert_instance_of User, article.author
  end

  test "has many comments" do
    article = articles(:published_article)
    assert_respond_to article, :comments
  end

  test "destroys comments when destroyed" do
    article = articles(:published_article)
    article.comments.create(body: "Test comment")

    assert_difference "Comment.count", -1 do
      article.destroy
    end
  end
end
```

## Testing Custom Methods

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  def published?
    published_at.present? && published_at <= Time.current
  end

  def reading_time
    words_per_minute = 200
    (body.split.size / words_per_minute.to_f).ceil
  end
end
```

```ruby
# test/models/article_test.rb
class ArticleTest < ActiveSupport::TestCase
  test "published? returns true for published articles" do
    article = Article.new(published_at: 1.day.ago)
    assert article.published?
  end

  test "published? returns false for future publish date" do
    article = Article.new(published_at: 1.day.from_now)
    assert_not article.published?
  end

  test "reading_time calculates correctly" do
    # 400 words should be 2 minutes
    article = Article.new(body: "word " * 400)
    assert_equal 2, article.reading_time
  end
end
```

## Testing Scopes

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  scope :published, -> { where("published_at <= ?", Time.current) }
  scope :recent, -> { order(created_at: :desc) }
end
```

```ruby
# test/models/article_test.rb
class ArticleTest < ActiveSupport::TestCase
  test "published scope returns only published articles" do
    published = Article.create(title: "Pub", body: "x", published_at: 1.day.ago)
    draft = Article.create(title: "Draft", body: "x", published_at: nil)

    results = Article.published
    assert_includes results, published
    assert_not_includes results, draft
  end
end
```

## Resources

- [Testing Rails Applications - Model Testing](https://guides.rubyonrails.org/testing.html#model-testing) — Guide to testing Rails models

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*