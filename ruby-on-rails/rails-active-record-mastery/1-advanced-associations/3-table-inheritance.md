---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-single-table-inheritance"
---

# Single Table Inheritance (STI)

Single Table Inheritance lets you store multiple model types in a single database table, sharing columns while having different behavior.

## When to Use STI

Use STI when models:
- Share most attributes
- Have different behavior (methods)
- Need to be queried together
- Don't have too many type-specific columns

## Example: Vehicle Types

```ruby
# app/models/vehicle.rb
class Vehicle < ApplicationRecord
  validates :brand, presence: true
  validates :model, presence: true

  def description
    "#{brand} #{model}"
  end
end

# app/models/car.rb
class Car < Vehicle
  validates :doors, presence: true

  def description
    "#{super} (#{doors}-door car)"
  end

  def honk
    "Beep beep!"
  end
end

# app/models/motorcycle.rb
class Motorcycle < Vehicle
  def description
    "#{super} (motorcycle)"
  end

  def rev
    "Vroom vroom!"
  end
end

# app/models/truck.rb
class Truck < Vehicle
  validates :payload_capacity, presence: true

  def description
    "#{super} (truck, #{payload_capacity}kg capacity)"
  end
end
```

## The Migration

All types share one table with a `type` column:

```ruby
class CreateVehicles < ActiveRecord::Migration[8.0]
  def change
    create_table :vehicles do |t|
      t.string :type, null: false  # STI column
      t.string :brand
      t.string :model
      t.integer :doors            # Car-specific
      t.integer :payload_capacity # Truck-specific
      t.string :color

      t.timestamps
    end

    add_index :vehicles, :type
  end
end
```

## Using STI

```ruby
# Create different types
car = Car.create(brand: "Toyota", model: "Camry", doors: 4)
moto = Motorcycle.create(brand: "Harley", model: "Sportster")
truck = Truck.create(brand: "Ford", model: "F-150", payload_capacity: 1000)

# Query all vehicles
Vehicle.all  # Returns Car, Motorcycle, and Truck instances

# Query specific types
Car.all           # Only cars
Motorcycle.all    # Only motorcycles

# Type-specific methods
car.honk          # => "Beep beep!"
moto.rev          # => "Vroom vroom!"

# Check type
car.type          # => "Car"
car.is_a?(Car)    # => true
car.is_a?(Vehicle) # => true
```

## STI with Associations

```ruby
class User < ApplicationRecord
  has_many :vehicles
  has_many :cars    # Only cars
  has_many :trucks  # Only trucks
end

# Usage
user.vehicles  # All vehicles
user.cars      # Just cars
```

## Pros and Cons

### Pros
- Simple to implement
- Easy to query across types
- No joins needed
- Shared validations and callbacks

### Cons
- Many NULL columns for type-specific attributes
- Table can grow large
- All types must fit in one table
- Not ideal for very different models

## Alternatives

- **Delegated Types** (Rails 6.1+): For more distinct models
- **Polymorphic Associations**: For flexible relationships
- **Separate Tables**: When models are truly different

## Resources

- [Active Record - Single Table Inheritance](https://api.rubyonrails.org/classes/ActiveRecord/Inheritance.html) — API documentation for STI

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*