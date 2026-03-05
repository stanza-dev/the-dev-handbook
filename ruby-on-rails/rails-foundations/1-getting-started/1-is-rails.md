---
source_course: "rails-foundations"
source_lesson: "rails-foundations-what-is-rails"
---

# What is Ruby on Rails?

## Introduction

Ruby on Rails is one of the most influential web frameworks ever created. If you want to build web applications quickly without reinventing the wheel, Rails gives you a battle-tested toolkit used by companies like GitHub, Shopify, and Airbnb.

## Key Concepts

- **Ruby on Rails (Rails)**: A server-side web application framework written in Ruby, created by David Heinemeier Hansson in 2004.
- **Convention over Configuration (CoC)**: Rails makes assumptions about what you want, so you write less configuration. A `User` model automatically maps to a `users` table.
- **Don't Repeat Yourself (DRY)**: Every piece of knowledge has a single, unambiguous representation in the system.
- **MVC (Model-View-Controller)**: The architectural pattern Rails follows to separate concerns.

## Real World Context

Rails powers some of the largest web applications in the world. Its "convention over configuration" philosophy means teams spend less time debating project structure and more time building features. Startups love Rails for rapid prototyping, while enterprises use it for its mature ecosystem.

## Deep Dive

### The MVC Architecture

Rails follows the Model-View-Controller pattern:

```
 Browser Request
       |
       v
     Router (config/routes.rb)
       |
       v
   Controller (app/controllers/)
   Handles requests, coordinates Model & View
     |              |
     v              v
   Model           View
 (app/models/)   (app/views/)
 Business logic   HTML templates
```

- **Model**: Handles data and business logic (talks to the database)
- **View**: Handles the presentation layer (HTML, JSON, etc.)
- **Controller**: Coordinates between Model and View, handles HTTP requests

### Convention over Configuration in Practice

```ruby
# A model called User automatically maps to a 'users' table
class User < ApplicationRecord
end

# A controller called UsersController looks for views in app/views/users/
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    # Automatically renders app/views/users/show.html.erb
  end
end
```

### Why Choose Rails?

| Feature | Benefit |
|---------|----------|
| **Rapid Development** | Build features in hours, not days |
| **Full-Stack** | Everything included: ORM, routing, views, testing |
| **Mature Ecosystem** | Thousands of gems (libraries) available |
| **Great Community** | Extensive documentation and support |
| **Battle-Tested** | Powers GitHub, Shopify, Airbnb, Basecamp |

## Common Pitfalls

- **Ignoring conventions**: Fighting Rails conventions leads to unnecessary complexity. Embrace them.
- **Skipping the docs**: Rails has excellent official guides; not reading them means missing powerful built-in features.
- **Over-engineering early**: Rails encourages starting simple; don't add complexity before you need it.

## Best Practices

- Follow Rails conventions for file naming, directory structure, and database schema.
- Read the official Rails Guides before reaching for third-party gems.
- Start with the default stack before customizing.

## Summary

- Rails is a full-stack web framework built on Ruby that emphasizes Convention over Configuration and DRY.
- It follows the MVC architecture: Models handle data, Views handle presentation, Controllers coordinate.
- Rails is battle-tested at scale and provides rapid development through its conventions and rich ecosystem.
- Companies like GitHub, Shopify, and Airbnb rely on Rails in production.

## Code Examples

**Rails conventions automatically map model classes to database tables and controllers to view directories, eliminating manual configuration.**

```ruby
# Convention over Configuration in action
class User < ApplicationRecord
  # Automatically maps to the 'users' table
  # Columns become attributes automatically
end

# UsersController looks for views in app/views/users/
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    # Renders app/views/users/show.html.erb by convention
  end
end
```


## Resources

- [Rails Doctrine](https://rubyonrails.org/doctrine) — The official Rails philosophy and core principles

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*