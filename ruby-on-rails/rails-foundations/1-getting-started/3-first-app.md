---
source_course: "rails-foundations"
source_lesson: "rails-foundations-creating-first-app"
---

# Creating Your First Rails Application

Let's create your first Rails application and explore its structure. This hands-on experience will help you understand how Rails organizes code.

## Creating a New Application

Open your terminal and run:

```bash
# Create a new Rails application called 'blog'
rails new blog

# Navigate into the application directory
cd blog
```

Rails generates a complete application structure:

```
blog/
├── app/                    # Main application code
│   ├── controllers/        # Handle HTTP requests
│   ├── models/             # Business logic & database
│   ├── views/              # HTML templates
│   ├── helpers/            # View helper methods
│   └── assets/             # CSS, images
├── config/                 # Configuration files
│   ├── routes.rb           # URL routing
│   ├── database.yml        # Database configuration
│   └── application.rb      # App settings
├── db/                     # Database files
│   ├── migrate/            # Database migrations
│   └── schema.rb           # Current schema
├── public/                 # Static files
├── test/                   # Test files
├── Gemfile                 # Ruby dependencies
└── Rakefile                # Task definitions
```

## Starting the Development Server

Rails includes a built-in web server called **Puma**:

```bash
# Start the server
bin/rails server

# Or use the shorthand
bin/rails s
```

You'll see output like:

```
=> Booting Puma
=> Rails 8.0.0 application starting in development
=> Run `bin/rails server --help` for more startup options
Puma starting in single mode...
* Listening on http://127.0.0.1:3000
```

Open **http://localhost:3000** in your browser. You should see the Rails welcome page!

## Understanding Key Files

### Gemfile
Lists all Ruby gems (libraries) your application uses:

```ruby
# Gemfile
source "https://rubygems.org"

gem "rails", "~> 8.0.0"
gem "sqlite3", ">= 2.1"
gem "puma", ">= 5.0"
```

After editing the Gemfile, run:
```bash
bundle install
```

### config/routes.rb
Defines how URLs map to controllers:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  # Define your routes here
  root "welcome#index"
end
```

### config/database.yml
Database configuration for each environment:

```yaml
# config/database.yml
default: &default
  adapter: sqlite3
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: storage/development.sqlite3

test:
  <<: *default
  database: storage/test.sqlite3

production:
  <<: *default
  database: storage/production.sqlite3
```

## The Rails Command Line

Rails provides many helpful commands:

```bash
# Generate code
bin/rails generate ...
bin/rails g ...          # Shorthand

# Database tasks
bin/rails db:migrate     # Run migrations
bin/rails db:seed        # Load seed data

# Console
bin/rails console        # Interactive Ruby with Rails loaded
bin/rails c              # Shorthand

# Server
bin/rails server         # Start development server
bin/rails s              # Shorthand
```

## Resources

- [Getting Started with Rails](https://guides.rubyonrails.org/getting_started.html) — Official guide to creating your first Rails application

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*