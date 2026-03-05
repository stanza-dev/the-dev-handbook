---
source_course: "rails-foundations"
source_lesson: "rails-foundations-directory-structure"
---

# Rails Directory Structure

## Introduction

When you run `rails new`, Rails generates a carefully organized directory structure. Understanding where files live and why is essential for navigating any Rails project confidently.

## Key Concepts

- **`app/`**: The heart of your application — models, views, controllers, helpers, mailers, and jobs live here.
- **`config/`**: Configuration files for routes, database, environments, and initializers.
- **`db/`**: Database migrations, schema file, and seed data.
- **`Gemfile`**: Declares all gem dependencies; Bundler uses it to install consistent versions.

## Real World Context

Every Rails developer needs to know the directory structure by heart. When a senior developer says "add a concern," you should immediately know to look in `app/models/concerns/`. When debugging a route, you go to `config/routes.rb`. This mental map saves time daily.

## Deep Dive

### The Top-Level Structure

Here is what `rails new myapp` generates:

```
myapp/
├── app/           # Application code (models, views, controllers)
├── bin/           # Scripts (rails, rake, setup)
├── config/        # Configuration files
├── db/            # Database files (migrations, schema, seeds)
├── lib/           # Custom library code and tasks
├── log/           # Application log files
├── public/        # Static files served directly by the web server
├── storage/       # Active Storage files (uploads)
├── test/          # Test files
├── tmp/           # Temporary files (cache, pids, sockets)
├── vendor/        # Third-party code
├── Gemfile        # Gem dependencies
├── Gemfile.lock   # Locked gem versions
└── Rakefile       # Rake task loader
```

Each directory has a specific purpose. Rails expects files in the right place — this is Convention over Configuration in action.

### Inside app/

The `app/` directory is where you spend most of your time:

```
app/
├── controllers/   # Handle HTTP requests
├── models/        # Business logic and database access
├── views/         # HTML templates (ERB, etc.)
├── helpers/       # View helper methods
├── mailers/       # Email sending classes
├── jobs/          # Background job classes
├── channels/      # Action Cable (WebSocket) channels
└── assets/        # CSS, JavaScript, images
```

Rails autoloads everything in `app/`, so you never need `require` statements for files here.

### Inside config/

The `config/` directory controls how your app behaves:

```ruby
# config/routes.rb — defines all URL routes
Rails.application.routes.draw do
  root "articles#index"
  resources :articles
end
```

The `routes.rb` file is one of the most frequently edited configuration files in any Rails app.

### Inside db/

The `db/` directory manages your database:

```
db/
├── migrate/       # Migration files (timestamped)
├── schema.rb      # Current database schema (auto-generated)
└── seeds.rb       # Seed data for development
```

The `schema.rb` file is auto-generated from your migrations. Never edit it manually.

## Common Pitfalls

- **Putting code in the wrong directory**: A service object belongs in `app/services/`, not `lib/`. Rails autoloads `app/` subdirectories but not `lib/` by default.
- **Editing `schema.rb` manually**: This file is auto-generated. Always create a migration instead.
- **Ignoring `config/environments/`**: Each environment (development, test, production) has different settings. Understand what each one configures.

## Best Practices

- Create custom subdirectories in `app/` for service objects (`app/services/`), form objects (`app/forms/`), etc. Rails autoloads them automatically.
- Keep `config/routes.rb` organized by resource. Use comments to separate sections in large apps.
- Run `bin/rails notes` to find TODO, FIXME, and OPTIMIZE comments across your codebase.

## Summary

- `app/` contains all application code: models, views, controllers, helpers, mailers, and jobs.
- `config/` holds configuration: routes, database settings, environment configs, and initializers.
- `db/` manages database migrations, the auto-generated schema, and seed data.
- Rails autoloads everything in `app/`, so no manual `require` statements are needed.
- Never edit `schema.rb` manually — always use migrations.

## Code Examples

**Key configuration files — routes.rb defines URL mappings, seeds.rb populates the database with initial data for development.**

```ruby
# config/routes.rb — the central routing file
Rails.application.routes.draw do
  root "articles#index"
  resources :articles
  resources :users, only: [:new, :create, :show]
end

# db/seeds.rb — populate development data
Article.create(title: "Welcome", body: "Hello Rails!")
User.create(email: "admin@example.com", password: "secret")
```


## Resources

- [Getting Started with Rails](https://guides.rubyonrails.org/getting_started.html) — Official guide covering the Rails directory structure and first application

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*