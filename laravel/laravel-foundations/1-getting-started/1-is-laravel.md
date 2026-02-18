---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-what-is-laravel"
---

# What is Laravel?

Laravel is a **web application framework** with expressive, elegant syntax. Created by Taylor Otwell in 2011, it has become the most popular PHP framework in the world. Laravel is designed to make development enjoyable and efficient while still being incredibly powerful.

## The Laravel Philosophy

Laravel is built on several key principles that make it unique:

### 1. Developer Happiness

Laravel prioritizes developer experience. Every feature is designed to be intuitive and expressive. The framework believes that happy developers write better code.

### 2. Convention Over Configuration

Laravel provides sensible defaults that work out of the box. You don't need to configure everything manually—the framework makes smart choices for you.

### 3. Don't Repeat Yourself (DRY)

Laravel helps you avoid duplication through:

- Reusable Blade components
- Eloquent models and relationships
- Service providers and dependency injection
- Built-in utilities and helpers

## The MVC Architecture

Laravel follows the **Model-View-Controller** pattern:

```
┌─────────────────────────────────────────────────────────┐
│                      HTTP Request                        │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                        Router                            │
│                    (routes/web.php)                      │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                      Controller                          │
│                (app/Http/Controllers)                    │
│         Handles requests, coordinates Model & View       │
└─────────────────────────────────────────────────────────┘
                     │              │
                     ▼              ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│          Model           │  │          View            │
│      (app/Models)        │  │    (resources/views)     │
│   Database & Logic       │  │   Blade templates        │
└──────────────────────────┘  └──────────────────────────┘
```

- **Model**: Represents your data and business logic (Eloquent ORM)
- **View**: Handles the presentation layer (Blade templates)
- **Controller**: Processes requests and returns responses

## Why Choose Laravel?

| Feature | Benefit |
|---------|----------|
| **Batteries Included** | Authentication, queues, caching, and more built-in |
| **Eloquent ORM** | Beautiful, intuitive database interactions |
| **Artisan CLI** | Powerful command-line tools |
| **Security First** | Protection against CSRF, XSS, SQL injection by default |
| **Scalable** | Powers Laracasts, Invoice Ninja, and millions of apps |
| **Great Ecosystem** | Forge, Vapor, Nova, Horizon, and more |
| **Amazing Documentation** | Clear, comprehensive, and well-maintained |
| **Active Community** | Millions of developers, regular updates |

## What You'll Build with Laravel

Laravel excels at building:

- **Web Applications** with rich user interfaces
- **REST APIs** for mobile and SPA frontends
- **E-commerce Platforms** with payment integration
- **Content Management Systems** (CMS)
- **Real-time Applications** with WebSockets
- **Microservices** and API backends

## Laravel's Ecosystem

Laravel has an incredible ecosystem of first-party tools:

- **Forge**: Server management and deployment
- **Vapor**: Serverless deployment on AWS
- **Nova**: Beautiful admin panels
- **Horizon**: Queue monitoring
- **Sanctum**: API authentication
- **Livewire**: Reactive components without JavaScript
- **Inertia**: Modern SPAs with Vue/React

## Resources

- [Laravel Documentation](https://laravel.com/docs/12.x) — Official Laravel 12 documentation - your primary reference

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*