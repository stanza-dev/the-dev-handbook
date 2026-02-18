---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-meta-options"
---

# Model Meta Options

The `Meta` class inside a model lets you define metadata that controls how Django handles your model. These options affect database behavior, admin display, and query defaults.

## Basic Meta Syntax

```python
from django.db import models


class Article(models.Model):
    title = models.CharField(max_length=200)
    pub_date = models.DateTimeField()
    author = models.ForeignKey('Author', on_delete=models.CASCADE)

    class Meta:
        ordering = ['-pub_date']  # Default ordering
        verbose_name = 'Article'
        verbose_name_plural = 'Articles'
```

## Database Table Options

### Custom Table Name

```python
class Meta:
    db_table = 'my_custom_articles'  # Default: appname_modelname
```

### Table Comment (Database-level)

```python
class Meta:
    db_table_comment = 'Stores all published articles'
```

## Ordering Options

### Default Ordering

```python
class Meta:
    # Single field
    ordering = ['title']
    
    # Descending order
    ordering = ['-pub_date']
    
    # Multiple fields
    ordering = ['-pub_date', 'title']
    
    # Order by expression
    ordering = [models.F('pub_date').desc(nulls_last=True)]
```

### Get Latest By

```python
class Meta:
    get_latest_by = 'pub_date'  # For .latest() and .earliest()
    # Or multiple fields:
    get_latest_by = ['-priority', 'pub_date']
```

Usage:
```python
Article.objects.latest()    # Most recent by pub_date
Article.objects.earliest()  # Oldest by pub_date
```

## Indexing

```python
class Meta:
    indexes = [
        models.Index(fields=['title']),
        models.Index(fields=['pub_date', 'author']),
        models.Index(fields=['-pub_date'], name='recent_articles_idx'),
        # Partial index (PostgreSQL)
        models.Index(
            fields=['title'],
            condition=models.Q(is_published=True),
            name='published_title_idx'
        ),
    ]
```

## Constraints

```python
class Meta:
    constraints = [
        # Unique constraint
        models.UniqueConstraint(
            fields=['author', 'title'],
            name='unique_author_title'
        ),
        # Check constraint
        models.CheckConstraint(
            check=models.Q(pub_date__lte=models.functions.Now()),
            name='pub_date_not_future'
        ),
    ]
```

## Display Options

```python
class Meta:
    verbose_name = 'news article'         # Singular name
    verbose_name_plural = 'news articles' # Plural name (admin)
```

## Abstract Models

Create base models that aren't created as tables:

```python
class TimestampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True  # No database table created


class Article(TimestampedModel):
    title = models.CharField(max_length=200)
    # Inherits created_at and updated_at
```

## Permissions

```python
class Meta:
    permissions = [
        ('can_publish', 'Can publish articles'),
        ('can_feature', 'Can feature on homepage'),
    ]
    default_permissions = ('add', 'change', 'delete', 'view')
```

## Other Useful Options

```python
class Meta:
    # Proxy model (same table, different behavior)
    proxy = True
    
    # Mark as managed by Django migrations
    managed = True  # False for legacy/external tables
    
    # Required fields for specific orderings
    order_with_respect_to = 'question'  # For related objects
```

## Resources

- [Model Meta Options](https://docs.djangoproject.com/en/6.0/ref/models/options/) — Complete reference for all Meta options

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*