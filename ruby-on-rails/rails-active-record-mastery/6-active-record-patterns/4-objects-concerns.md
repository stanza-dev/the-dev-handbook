---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-query-objects-concerns"
---

# Query Objects and Concerns

## Introduction
As Rails applications grow, models accumulate scopes, query methods, and shared behavior. `ActiveSupport::Concern` provides a clean way to extract reusable model modules. Combined with query objects, concerns help you organize large models into focused, testable pieces.

## Key Concepts
- **`ActiveSupport::Concern`**: A Rails module mixin that provides `included` and `class_methods` blocks for cleanly adding behavior to models.
- **Query object**: A plain Ruby class dedicated to building a single complex query.
- **Model concern**: A module that extracts a cohesive set of scopes, validations, callbacks, or methods from a model.
- **`included` block**: Code inside this block runs in the context of the including class (for validations, scopes, callbacks).

## Real World Context
A `User` model in a large application might have authentication logic, profile methods, notification preferences, subscription management, and admin features. Without concerns, this model grows to 500+ lines. Concerns like `Authenticatable`, `Subscribable`, and `Notifiable` split it into manageable pieces, each testable in isolation.

## Deep Dive

### ActiveSupport::Concern

```ruby
# app/models/concerns/searchable.rb
module Searchable
  extend ActiveSupport::Concern

  included do
    scope :search, ->(query) {
      return all if query.blank?
      where("name ILIKE :q OR description ILIKE :q", q: "%#{query}%")
    }
  end

  class_methods do
    def most_searched
      order(search_count: :desc).limit(10)
    end
  end

  def matches_search?(query)
    name.downcase.include?(query.downcase) ||
      description&.downcase&.include?(query.downcase)
  end
end

# Usage in models
class Product < ApplicationRecord
  include Searchable
end

class Article < ApplicationRecord
  include Searchable
end

# Now both models have .search, .most_searched, and #matches_search?
```

The `included` block is where you put scopes, validations, and callbacks. The `class_methods` block defines class-level methods. Instance methods go directly in the module body.

### Common Concern Patterns

#### Soft Delete Concern

```ruby
# app/models/concerns/soft_deletable.rb
module SoftDeletable
  extend ActiveSupport::Concern

  included do
    scope :active, -> { where(deleted_at: nil) }
    scope :deleted, -> { where.not(deleted_at: nil) }

    default_scope { active }
  end

  def soft_delete!
    update!(deleted_at: Time.current)
  end

  def restore!
    update!(deleted_at: nil)
  end

  def deleted?
    deleted_at.present?
  end
end
```

#### Sluggable Concern

```ruby
# app/models/concerns/sluggable.rb
module Sluggable
  extend ActiveSupport::Concern

  included do
    before_validation :generate_slug, if: -> { slug.blank? && respond_to?(:name) }
    validates :slug, presence: true, uniqueness: true
  end

  private

  def generate_slug
    self.slug = name.parameterize if name.present?
  end
end
```

### Advanced Query Objects

Query objects can also compose with concerns:

```ruby
# app/queries/filterable_query.rb
class FilterableQuery
  def initialize(relation)
    @relation = relation
  end

  def call(filters = {})
    filters.each do |key, value|
      next if value.blank?
      @relation = apply_filter(@relation, key, value)
    end
    @relation
  end

  private

  def apply_filter(relation, key, value)
    case key.to_sym
    when :search
      relation.where("name ILIKE ?", "%#{value}%")
    when :status
      relation.where(status: value)
    when :created_after
      relation.where("created_at >= ?", value)
    when :created_before
      relation.where("created_at <= ?", value)
    when :sort
      apply_sort(relation, value)
    else
      relation
    end
  end

  def apply_sort(relation, sort)
    case sort
    when "newest" then relation.order(created_at: :desc)
    when "oldest" then relation.order(created_at: :asc)
    when "name" then relation.order(:name)
    else relation
    end
  end
end
```

## Common Pitfalls
1. **Concerns that depend on each other** -- If ConcernA requires ConcernB, the model must include both in the right order. Keep concerns independent.
2. **Using concerns as code dumping grounds** -- A concern should represent a cohesive concept (searchable, taggable, auditable), not just "methods I moved out of the model".
3. **Too many concerns on one model** -- If a model includes 10+ concerns, the complexity is just hidden, not reduced. Consider whether the model has too many responsibilities.

## Best Practices
1. **Name concerns as adjectives** -- `Searchable`, `Taggable`, `SoftDeletable` clearly describe the behavior they add.
2. **Keep concerns focused** -- Each concern should add one cohesive capability.
3. **Test concerns independently** -- Include the concern in a test model (or use shared examples) so you can test it without the full model.

## Summary
- `ActiveSupport::Concern` provides `included` and `class_methods` blocks for clean model mixins.
- Use concerns to extract cohesive behavior: searching, soft-deleting, slugging, auditing.
- Query objects handle complex multi-parameter queries that outgrow scopes.
- Name concerns as adjectives and keep them focused on a single responsibility.
- Test concerns independently using shared examples or test models.

## Code Examples

**An ActiveSupport::Concern that adds search functionality -- any model that includes it gets the search scope and class methods**

```ruby
# app/models/concerns/searchable.rb
module Searchable
  extend ActiveSupport::Concern

  included do
    scope :search, ->(query) {
      return all if query.blank?
      where("name ILIKE :q OR description ILIKE :q", q: "%#{query}%")
    }
  end

  class_methods do
    def most_searched
      order(search_count: :desc).limit(10)
    end
  end
end

# Any model can include this concern
class Product < ApplicationRecord
  include Searchable
end
```


## Resources

- [ActiveSupport::Concern](https://api.rubyonrails.org/classes/ActiveSupport/Concern.html) — API documentation for ActiveSupport::Concern

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*