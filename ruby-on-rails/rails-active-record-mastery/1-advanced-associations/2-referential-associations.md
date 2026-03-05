---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-self-referential-associations"
---

# Self-Referential Associations

## Introduction
Self-referential associations connect a model to itself. They appear whenever you need hierarchies (employee/manager), social graphs (followers/following), or nested structures (categories with subcategories). Understanding them unlocks some of the most common data modeling patterns.

## Key Concepts
- **Self-referential association**: An association where the foreign key points back to the same table.
- **`class_name`**: Tells Rails which model class to use when the association name does not match the class name.
- **`foreign_key`**: Specifies which column holds the reference back to the same table.
- **`source`**: Used with `through` associations to identify which association to follow on the join model.

## Real World Context
Social networks use self-referential associations for followers and following. Organizational charts model employee-manager hierarchies. E-commerce platforms use them for nested categories. Any tree or graph structure stored in a relational database typically relies on self-referential associations.

## Deep Dive

### Employee/Manager Hierarchy

An employee belongs to a manager, who is also an employee:

```ruby
# app/models/employee.rb
class Employee < ApplicationRecord
  belongs_to :manager, class_name: "Employee", optional: true
  has_many :subordinates, class_name: "Employee", foreign_key: "manager_id"
end
```

The `class_name: "Employee"` tells Rails that `:manager` and `:subordinates` both refer to the Employee model, not a separate Manager or Subordinate model.

#### The Migration

```ruby
class CreateEmployees < ActiveRecord::Migration[8.1]
  def change
    create_table :employees do |t|
      t.string :name
      t.references :manager, foreign_key: { to_table: :employees }
      t.timestamps
    end
  end
end
```

Note the `to_table: :employees` option, which tells the foreign key constraint to point back to the same table.

#### Usage

```ruby
ceo = Employee.create(name: "Alice", manager: nil)
vp = Employee.create(name: "Bob", manager: ceo)
dev = Employee.create(name: "Charlie", manager: vp)

ceo.subordinates    # => [Bob]
vp.manager          # => Alice
vp.subordinates     # => [Charlie]
dev.manager         # => Bob
```

### Social Network Followers

Users follow other users (many-to-many self-referential):

```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_many :active_follows, class_name: "Follow",
                            foreign_key: "follower_id",
                            dependent: :destroy
  has_many :following, through: :active_follows, source: :followed

  has_many :passive_follows, class_name: "Follow",
                             foreign_key: "followed_id",
                             dependent: :destroy
  has_many :followers, through: :passive_follows, source: :follower

  def follow(other_user)
    following << other_user unless self == other_user
  end

  def unfollow(other_user)
    following.delete(other_user)
  end

  def following?(other_user)
    following.include?(other_user)
  end
end

# app/models/follow.rb
class Follow < ApplicationRecord
  belongs_to :follower, class_name: "User"
  belongs_to :followed, class_name: "User"
  validates :follower_id, uniqueness: { scope: :followed_id }
end
```

The `source:` option is essential here: it tells Rails which association on the Follow model to use when traversing the `through` relationship.

#### The Migration

```ruby
class CreateFollows < ActiveRecord::Migration[8.1]
  def change
    create_table :follows do |t|
      t.references :follower, null: false, foreign_key: { to_table: :users }
      t.references :followed, null: false, foreign_key: { to_table: :users }
      t.timestamps
    end
    add_index :follows, [:follower_id, :followed_id], unique: true
  end
end
```

## Common Pitfalls
1. **Forgetting `class_name`** -- Without it, Rails looks for a `Manager` or `Subordinate` model that does not exist and raises a `NameError`.
2. **Infinite recursion in serialization** -- Calling `.to_json` on a self-referential tree without limiting depth can cause stack overflow. Always use `as_json(only: ...)` or a serializer with depth control.
3. **Missing `optional: true`** -- In a hierarchy, the root node has no parent. Without `optional: true`, validation will fail for root records.

## Best Practices
1. **Use `class_name` and `foreign_key` explicitly** -- Even when Rails could infer them, being explicit makes the code self-documenting.
2. **Add database-level unique indexes** -- For join tables like follows, add a composite unique index to prevent duplicate relationships.
3. **Consider ancestry gems for deep trees** -- For deeply nested hierarchies (categories 5+ levels deep), gems like `ancestry` or `closure_tree` provide efficient tree queries.

## Summary
- Self-referential associations use `class_name` to point a model back to itself.
- Use `foreign_key` and `source` to disambiguate when association names differ from class names.
- Many-to-many self-joins require a join model (like Follow) with two foreign keys to the same table.
- Always add `optional: true` for root nodes in hierarchies.
- For deep trees, consider specialized gems like `ancestry` or `closure_tree`.

## Code Examples

**A self-referential employee/manager hierarchy -- both sides of the relationship point to the Employee model**

```ruby
class Employee < ApplicationRecord
  # Points back to the same table
  belongs_to :manager, class_name: "Employee", optional: true
  has_many :subordinates, class_name: "Employee", foreign_key: "manager_id"
end

ceo = Employee.create(name: "Alice")
vp = Employee.create(name: "Bob", manager: ceo)

ceo.subordinates  # => [#<Employee name: "Bob">]
vp.manager        # => #<Employee name: "Alice">
```


## Resources

- [Active Record Associations - Self Joins](https://guides.rubyonrails.org/association_basics.html#self-joins) — Official guide to self-referential associations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*