---
source_course: "rails-foundations"
source_lesson: "rails-foundations-creating-first-app"
---

# Creating Your First Rails Application

## Introduction

The best way to learn Rails is to build something. In just a few commands you can generate a complete application skeleton and see it running in your browser.

## Key Concepts

- **`rails new`**: The command that generates a complete Rails application with all the boilerplate code and directory structure.
- **Application structure**: Rails organizes code into `app/` (your code), `config/` (settings), `db/` (database), and `test/` (tests).
- **Puma**: The built-in web server that serves your Rails application in development.
- **Gemfile**: Lists all Ruby gem dependencies for your project, managed by Bundler.

## Real World Context

Every Rails project starts with `rails new`. Understanding the generated directory structure is essential because Rails relies on files being in specific locations. This convention-based structure is what makes Rails teams productive — any Rails developer can jump into a new project and immediately know where to find things.

## Deep Dive

### Creating a New Application

```bash
rails new blog
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
├── test/                   # Test files
├── Gemfile                 # Ruby dependencies
└── Rakefile                # Task definitions
```

### Starting the Development Server

```bash
bin/rails server
```

Output:

```
=> Booting Puma
=> Rails 8.1.0 application starting in development
=> Run `bin/rails server --help` for more startup options
Puma starting in single mode...
* Listening on http://127.0.0.1:3000
```

### Understanding Key Files

#### Gemfile

```ruby
source "https://rubygems.org"

gem "rails", "~> 8.1.0"
gem "sqlite3", ">= 2.1"
gem "puma", ">= 5.0"
```

#### config/routes.rb

```ruby
Rails.application.routes.draw do
  root "welcome#index"
end
```

### The Rails Command Line

```bash
bin/rails generate ...   # Generate code
bin/rails db:migrate     # Run migrations
bin/rails console        # Interactive Ruby with Rails loaded
bin/rails server         # Start development server
```

## Common Pitfalls

- **Not running `bundle install`** after editing the Gemfile. Always run it to install new gems.
- **Editing `db/schema.rb` directly**: This file is auto-generated. Use migrations to change the database schema.
- **Forgetting to start the server**: After `rails new`, you must run `bin/rails server` to see your app.

## Best Practices

- Commit your generated app to Git immediately after `rails new`.
- Use `bin/rails` instead of `rails` to ensure you use the project's bundled version.
- Explore the generated files to understand how Rails organizes code.

## Summary

- `rails new <name>` generates a complete application skeleton.
- The `app/` directory contains your models, views, and controllers.
- `bin/rails server` starts the development server on port 3000.
- The Gemfile manages dependencies; `config/routes.rb` maps URLs to controllers.
- Use `bin/rails console` for interactive exploration of your app.

## Code Examples

**The Gemfile lists your application's gem dependencies, while config/routes.rb defines how URLs map to controller actions.**

```ruby
# Gemfile - Ruby dependencies
source "https://rubygems.org"

gem "rails", "~> 8.1.0"
gem "sqlite3", ">= 2.1"
gem "puma", ">= 5.0"

# config/routes.rb - URL routing
Rails.application.routes.draw do
  root "welcome#index"
end
```


## Resources

- [Getting Started with Rails](https://guides.rubyonrails.org/getting_started.html) — Official guide to creating your first Rails application

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*