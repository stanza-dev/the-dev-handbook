---
source_course: "rails-foundations"
source_lesson: "rails-foundations-what-is-rails"
---

# What is Ruby on Rails?

Ruby on Rails (often just called "Rails") is a **server-side web application framework** written in Ruby. Created by David Heinemeier Hansson in 2004, Rails has become one of the most influential frameworks in web development history.

## The Rails Philosophy

Rails is built on two core principles that make it unique:

### 1. Convention over Configuration (CoC)

Rails makes assumptions about what you want to do and how you're going to do it. Instead of configuring everything manually, you follow conventions:

- A model called `User` automatically maps to a database table called `users`
- A controller called `UsersController` automatically looks for views in `app/views/users/`
- URLs like `/users/1` automatically route to the `show` action with `id: 1`

This means **less code, fewer decisions, and faster development**.

### 2. Don't Repeat Yourself (DRY)

Every piece of knowledge should have a single, unambiguous representation in a system. Rails helps you avoid duplication through:

- Shared layouts and partials
- Helper methods
- Concerns for shared functionality
- Migrations that define schema in one place

## The MVC Architecture

Rails follows the **Model-View-Controller** pattern:

```
┌─────────────────────────────────────────────────────────┐
│                      Browser Request                      │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                        Router                            │
│              (config/routes.rb)                          │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                      Controller                          │
│              (app/controllers/)                          │
│         Handles requests, coordinates Model & View       │
└─────────────────────────────────────────────────────────┘
                     │              │
                     ▼              ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│          Model           │  │           View           │
│      (app/models/)       │  │       (app/views/)       │
│   Business logic, DB     │  │    HTML templates        │
└──────────────────────────┘  └──────────────────────────┘
```

- **Model**: Handles data and business logic (talks to the database)
- **View**: Handles the presentation layer (HTML, JSON, etc.)
- **Controller**: Coordinates between Model and View, handles HTTP requests

## Why Choose Rails?

| Feature | Benefit |
|---------|----------|
| **Rapid Development** | Build features in hours, not days |
| **Full-Stack** | Everything included: ORM, routing, views, testing |
| **Mature Ecosystem** | Thousands of gems (libraries) available |
| **Great Community** | Extensive documentation and support |
| **Battle-Tested** | Powers GitHub, Shopify, Airbnb, Basecamp |

## What You'll Build with Rails

Rails excels at building:

- **Web applications** with user authentication
- **APIs** for mobile apps and SPAs
- **E-commerce platforms**
- **Content management systems**
- **Real-time applications** with WebSockets

## Resources

- [Rails Doctrine](https://rubyonrails.org/doctrine) — The official Rails philosophy and core principles

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*