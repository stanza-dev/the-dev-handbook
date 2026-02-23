---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-inheritance-patterns"
---

# Model Inheritance Patterns

## Introduction

Django supports three model inheritance strategies, each with different database implications. Choosing the right one affects query performance, schema complexity, and code reuse.

## Key Concepts

- **Abstract Base Classes**: Share fields across models without creating a database table.
- **Multi-Table Inheritance**: Each model gets its own table, linked by implicit OneToOneField.
- **Proxy Models**: Add Python-level behavior without changing the database schema.

## Real World Context

In e-commerce, you might have `Product` as a base with `DigitalProduct` and `PhysicalProduct` as subclasses. Choosing abstract vs multi-table determines whether you query one table or join across three.

## Deep Dive

### Abstract Base Classes

Abstract base classes let you define shared fields in a parent model without creating a database table. All fields are copied into each child model's table:

```python
class TimestampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True  # No database table created

class Article(TimestampedModel):
    title = models.CharField(max_length=200)
    # Gets created_at + updated_at columns in article table

class Comment(TimestampedModel):
    text = models.TextField()
    # Gets created_at + updated_at columns in comment table
```

Notice `abstract = True` in the Meta class — this  not to create a table for TimestampedModel. Both Article and Comment will have their own `created_at` and `updated_at` columns directly in their tables. No JOIN needed.

### Multi-Table Inheritance

With multi-table inheritance, each model in the hierarchy gets its own database table. Django creates an implicit OneToOneField linking child to parent:

```python
class Place(models.Model):
    name = models.CharField(max_length=100)
    address = models.TextField()

class Restaurant(Place):
    cuisine = models.CharField(max_length=50)
    serves_pizza = models.BooleanField(default=False)
    # Implicit OneToOneField to Place

# Querying:
restaurant = Restaurant.objects.get(pk=1)
print(restaurant.name)  # Accessed via JOIN to place table
print(restaurant.cuisine)  # Direct column
```

Django creates a separate `restaurant` table with an implicit `place_ptr` OneToOneField. When you access `restaurant.name`, Django performs a JOIN to the `place` table. This means every Restaurant query is slightly slower than a single-table query.

### Proxy Models

Proxy models share the exact same database table as their parent but let you change Python-level behavior like ordering, managers, or methods:

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    pub_date = models.DateTimeField(null=True)

class PublishedArticle(Article):
    class Meta:
        proxy = True  # Same table, different Python class
        ordering = ['-pub_date']

    objects = PublishedManager()  # Custom manager

    def is_recent(self):
        return self.pub_date >= timezone.now() - timedelta(days=7)
```

Both `Article` and `PublishedArticle` read from and write to the same database table. The proxy simply provides a different Python interface — different default ordering, a different manager, and the `is_recent()` helper method.

## Common Pitfalls

1. **Using multi-table inheritance when abstract would suffice** — Multi-table adds JOINs on every query. Only use it when you need to query the parent table independently.
2. **Adding fields to proxy models** — Proxy models cannot add new database fields. They only add Python behavior.
3. **Forgetting `abstract = True`** — Without it, Django creates a table for the base class and uses multi-table inheritance.

## Best Practices

1. **Default to abstract base classes** — They're the simplest and most performant option for sharing fields.
2. **Use proxy models for alternate interfaces** — Different managers, ordering, or methods on the same data.
3. **Reserve multi-table for truly polymorphic data** — When you need `Place.objects.all()` to return both restaurants and cafes.

## Summary

- Abstract base classes copy fields to children — no JOINs, no parent table.
- Multi-table inheritance creates separate tables joined by OneToOneField.
- Proxy models share the parent table but can have different Python behavior.
- Prefer abstract models for code reuse; use multi-table only when querying the parent independently.

## Code Examples

**Abstract base class providing timestamp fields — no JOIN, fields are copied to each child table**

```python
class TimestampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True

class Article(TimestampedModel):
    title = models.CharField(max_length=200)
    # Inherits created_at and updated_at
```


## Resources

- [Model Inheritance](https://docs.djangoproject.com/en/6.0/topics/db/models/#model-inheritance) — Official Django guide to model inheritance strategies

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*