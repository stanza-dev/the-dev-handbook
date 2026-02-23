---
source_course: "django-foundations"
source_lesson: "django-foundations-what-is-django"
---

# What is Django?

Django is a **high-level Python web framework** that encourages rapid development and clean, pragmatic design. Created in 2003 by Adrian Holovaty and Simon Willison at the Lawrence Journal-World newspaper, Django was released publicly in 2005 and has since become one of the most popular web frameworks in the world.

## The Django Philosophy

Django is built on several key principles that make it unique and powerful:

### 1. Don't Repeat Yourself (DRY)

Every piece of knowledge should have a single, unambiguous representation in a system. Django helps you avoid duplication through:

- Reusable applications
- Template inheritance
- Model abstractions
- Built-in utilities

### 2. Explicit is Better Than Implicit

Django follows Python's philosophy: code should be clear and readable. Magic is minimized in favor of explicit declarations.

### 3. Loose Coupling

Components in Django are designed to be independent. The template system doesn't need to know about the database, and views don't need to know about templates.

## The MTV Architecture

Django follows the **Model-Template-View** pattern (similar to MVC):

```
┌─────────────────────────────────────────────────────────┐
│                      Browser Request                     │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                      URL Dispatcher                      │
│                    (urls.py)                             │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                         View                             │
│                    (views.py)                            │
│         Handles requests, coordinates Model & Template   │
└─────────────────────────────────────────────────────────┘
                     │              │
                     ▼              ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│          Model           │  │         Template         │
│       (models.py)        │  │       (*.html)           │
│   Database & Logic       │  │    HTML presentation     │
└──────────────────────────┘  └──────────────────────────┘
```

- **Model**: Defines your data structure and handles database interactions
- **Template**: Handles the presentation layer (HTML rendering)
- **View**: Contains the business logic, processes requests, and returns responses

## Why Choose Django?

| Feature | Benefit |
|---------|----------|
| **Batteries Included** | Authentication, admin, ORM, and more built-in |
| **Security First** | Protection against CSRF, XSS, SQL injection by default |
| **Scalable** | Powers Instagram, Pinterest, Mozilla, Disqus |
| **Versatile** | Build websites, APIs, CMS, or scientific platforms |
| **Great Documentation** | One of the best-documented frameworks |
| **Active Community** | Regular updates and thousands of packages |

## What You'll Build with Django

Django excels at building:

- **Content Management Systems** (CMS)
- **Social networks and community sites**
- **E-commerce platforms**
- **REST APIs** for mobile applications
- **Scientific computing platforms**
- **News and publishing sites**

## Common Pitfalls

- **Confusing MTV with MVC**: Django's View is like MVC's Controller, and Django's Template is like MVC's View. The naming difference trips up developers coming from other frameworks.
- **Assuming Django is only for large projects**: Django's "batteries included" philosophy works great for small projects too. You can ignore features you don't need.
- **Skipping the official tutorial**: Django's documentation is exceptionally good. Jumping straight into third-party tutorials often means missing important conventions.

## Best Practices

- **Follow Django conventions**: Use the standard project structure and naming patterns. Fighting the framework leads to frustration.
- **Keep apps small and focused**: Each app should do one thing well. This promotes reusability and clean architecture.
- **Read the release notes**: Django's release notes document breaking changes, deprecations, and new features clearly.

## Summary

- Django is a high-level Python web framework following the **Model-Template-View (MTV)** architectural pattern
- Core principles include DRY (Don't Repeat Yourself), explicit over implicit, and loose coupling
- Django is "batteries included" with built-in authentication, admin, ORM, and security protections
- It powers major sites like Instagram and Pinterest, proving its scalability
- The framework excels at content management systems, APIs, e-commerce platforms, and social networks

## Code Examples

**A simple Django view function that returns an HTTP response, demonstrating the core request-response cycle**

```python
# A minimal Django view
from django.http import HttpResponse

def hello(request):
    return HttpResponse("Hello, Django!")
```


## Resources

- [Django Overview](https://docs.djangoproject.com/en/6.0/intro/overview/) — Official overview of Django's capabilities and design philosophy

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*