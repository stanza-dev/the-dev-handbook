---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-self-referential-associations"
---

# Self-Referential Associations

Self-referential associations connect a model to itself. Common examples include followers/following, employees and managers, or nested categories.

## Example: Employee/Manager Hierarchy

An employee belongs to a manager, who is also an employee:

```ruby
# app/models/employee.rb
class Employee < ApplicationRecord
  # Manager relationship
  belongs_to :manager, class_name: "Employee", optional: true

  # Subordinates relationship
  has_many :subordinates, class_name: "Employee", foreign_key: "manager_id"
end
```

### The Migration

```ruby
class CreateEmployees < ActiveRecord::Migration[8.0]
  def change
    create_table :employees do |t|
      t.string :name
      t.references :manager, foreign_key: { to_table: :employees }

      t.timestamps
    end
  end
end
```

### Usage

```ruby
ceo = Employee.create(name: "Alice", manager: nil)
vp = Employee.create(name: "Bob", manager: ceo)
dev = Employee.create(name: "Charlie", manager: vp)

ceo.subordinates    # => [Bob]
vp.manager          # => Alice
vp.subordinates     # => [Charlie]
dev.manager         # => Bob
```

## Example: Social Network Followers

Users follow other users (many-to-many self-referential):

```ruby
# app/models/user.rb
class User < ApplicationRecord
  # Following relationships
  has_many :active_follows, class_name: "Follow",
                            foreign_key: "follower_id",
                            dependent: :destroy
  has_many :following, through: :active_follows, source: :followed

  # Follower relationships
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

### The Migration

```ruby
class CreateFollows < ActiveRecord::Migration[8.0]
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

### Usage

```ruby
alice = User.create(name: "Alice")
bob = User.create(name: "Bob")
charlie = User.create(name: "Charlie")

alice.follow(bob)
alice.follow(charlie)
bob.follow(alice)

alice.following      # => [Bob, Charlie]
alice.followers      # => [Bob]
bob.following        # => [Alice]
bob.followers        # => [Alice]

alice.following?(bob)      # => true
alice.following?(charlie)  # => true
bob.following?(charlie)    # => false
```

## Key Points

- Use `class_name` to specify the model when the association name differs
- Use `foreign_key` to specify which column holds the reference
- Use `source` with `through` associations to specify the source association name

## Resources

- [Active Record Associations - Self Joins](https://guides.rubyonrails.org/association_basics.html#self-joins) — Official guide to self-referential associations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*