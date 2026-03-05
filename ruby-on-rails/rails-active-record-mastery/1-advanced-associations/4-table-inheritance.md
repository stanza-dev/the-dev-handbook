---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-single-table-inheritance"
---

# Single Table Inheritance (STI)

## Introduction
Single Table Inheritance lets you store multiple model types in a single database table, sharing columns while having different behavior. It is one of the oldest patterns in Active Record and remains useful when your subtypes share most attributes but differ in methods and validations.

## Key Concepts
- **STI**: A pattern where subclasses inherit from a base model and all share the same database table.
- **`type` column**: A string column that Rails uses to store the subclass name and instantiate the correct class.
- **`inheritance_column`**: The Active Record setting that controls which column stores the type (defaults to `"type"`).

## Real World Context
STI is commonly used for notification types (EmailNotification, SMSNotification, PushNotification), payment methods (CreditCard, BankTransfer, PayPal), or content types (Article, Video, Podcast) when they share 80%+ of their attributes. If your subtypes diverge significantly, consider delegated types or separate tables instead.

## Deep Dive

### Defining STI Models

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

All types share the `vehicles` table. Rails stores the class name in the `type` column and uses it to instantiate the correct class when loading records.

### The Migration

```ruby
class CreateVehicles < ActiveRecord::Migration[8.1]
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

Note the `type` column with a `null: false` constraint. The index on `type` speeds up queries scoped to a single subclass.

### Using STI

```ruby
car = Car.create(brand: "Toyota", model: "Camry", doors: 4)
moto = Motorcycle.create(brand: "Harley", model: "Sportster")
truck = Truck.create(brand: "Ford", model: "F-150", payload_capacity: 1000)

# Query all vehicles
Vehicle.all  # Returns Car, Motorcycle, and Truck instances

# Query specific types
Car.all           # Only cars (WHERE type = 'Car')
Motorcycle.all    # Only motorcycles

# Type-specific methods
car.honk          # => "Beep beep!"
moto.rev          # => "Vroom vroom!"

# Check type
car.type          # => "Car"
car.is_a?(Car)    # => true
car.is_a?(Vehicle) # => true
```

### Disabling STI

If you have a column named `type` that is not intended for STI, you can disable it:

```ruby
class Event < ApplicationRecord
  self.inheritance_column = nil  # Disables STI
  # Now 'type' is treated as a regular string column
end
```

This is a common gotcha: adding a `type` column for a different purpose accidentally activates STI behavior. Setting `self.inheritance_column = nil` prevents this.

## Common Pitfalls
1. **Too many NULL columns** -- Type-specific columns (like `doors` for Car, `payload_capacity` for Truck) are NULL for all other types. If each subclass has many unique columns, the table becomes sparse and wasteful.
2. **Accidental STI activation** -- Adding a `type` column for business logic (e.g., event types) triggers STI. Use `self.inheritance_column = nil` to disable it.
3. **Overusing STI for divergent models** -- If subtypes share fewer than 80% of their columns, separate tables or delegated types are a better choice.

## Best Practices
1. **Index the type column** -- All subclass queries filter by type, so an index is essential for performance.
2. **Keep subtypes similar** -- STI works best when subtypes share most attributes and differ mainly in behavior.
3. **Consider delegated types for complex hierarchies** -- Rails 6.1+ delegated types offer a middle ground between STI and separate tables.

## Summary
- STI stores multiple model types in one table using a `type` column.
- Subclasses inherit from the base model and can add their own validations and methods.
- Use `self.inheritance_column = nil` to disable STI when you need a `type` column for other purposes.
- STI is best when subtypes share most attributes; use delegated types or separate tables for divergent models.
- Always index the `type` column for query performance.

## Code Examples

**STI in action -- Car and Motorcycle inherit from Vehicle and share the same database table, differentiated by the `type` column**

```ruby
class Vehicle < ApplicationRecord
  validates :brand, presence: true
end

class Car < Vehicle
  def honk
    "Beep beep!"
  end
end

class Motorcycle < Vehicle
  def rev
    "Vroom vroom!"
  end
end

# All stored in the 'vehicles' table
Car.create(brand: "Toyota")
Vehicle.all  # => [#<Car brand: "Toyota">]
Car.first.type  # => "Car"
```


## Resources

- [Active Record - Single Table Inheritance](https://api.rubyonrails.org/classes/ActiveRecord/Inheritance.html) — API documentation for STI

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*