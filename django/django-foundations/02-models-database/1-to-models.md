---
source_course: "django-foundations"
source_lesson: "django-foundations-introduction-to-models"
---

# Introduction to Django Models

Models are Python classes that define the structure of your data. Django's **Object-Relational Mapper (ORM)** translates these Python classes into database tables, letting you work with data using Python instead of SQL.

## What is an ORM?

An ORM maps:
- **Python classes** → Database tables
- **Class attributes** → Table columns
- **Class instances** → Table rows

```
Python Class: Person              Database Table: myapp_person
┌─────────────────────────┐       ┌────┬────────────┬───────────┐
│ class Person(Model):    │       │ id │ first_name │ last_name │
│   first_name = Char...  │  ───► ├────┼────────────┼───────────┤
│   last_name = Char...   │       │ 1  │ "John"     │ "Doe"     │
└─────────────────────────┘       │ 2  │ "Jane"     │ "Smith"   │
                                  └────┴────────────┴───────────┘
```

## Creating Your First Model

Let's create models for our polls app:

```python
# polls/models.py
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

## Understanding the Code

### Model Class

Each model is a Python class that subclasses `django.db.models.Model`:

```python
class Question(models.Model):  # Inherits from Model
    pass
```

### Fields

Each class attribute represents a database column:

```python
question_text = models.CharField(max_length=200)
#     │                   │              │
#  column name       field type     field options
```

### Field Types

Django provides many field types:

| Field Type | Python Type | Description |
|------------|-------------|-------------|
| `CharField` | `str` | Short text (requires `max_length`) |
| `TextField` | `str` | Long text (unlimited) |
| `IntegerField` | `int` | Whole numbers |
| `FloatField` | `float` | Floating point numbers |
| `DecimalField` | `Decimal` | Precise decimals |
| `BooleanField` | `bool` | True/False |
| `DateField` | `date` | Date only |
| `DateTimeField` | `datetime` | Date and time |
| `EmailField` | `str` | Validated email |
| `URLField` | `str` | Validated URL |
| `ForeignKey` | Model instance | Many-to-one relationship |

### Field Options

Common options for all field types:

```python
# Required vs optional
name = models.CharField(max_length=100)  # Required by default
bio = models.TextField(blank=True)        # Optional (can be empty)

# Default values
votes = models.IntegerField(default=0)
created = models.DateTimeField(auto_now_add=True)

# Null values
middle_name = models.CharField(max_length=50, null=True, blank=True)

# Human-readable name
pub_date = models.DateTimeField('date published')
```

### Relationships

The `ForeignKey` creates a many-to-one relationship:

```python
question = models.ForeignKey(Question, on_delete=models.CASCADE)
#              │                  │                    │
#         field type        related model      deletion behavior
```

`on_delete` options:
- `CASCADE`: Delete related objects when parent is deleted
- `PROTECT`: Prevent deletion if related objects exist
- `SET_NULL`: Set to NULL (requires `null=True`)
- `SET_DEFAULT`: Set to default value

## Common Pitfalls

- **Forgetting `max_length` on CharField**: `CharField` requires a `max_length` argument. Without it, Django raises an error during validation.
- **Using `null=True` on string fields**: For `CharField` and `TextField`, use `blank=True` instead of `null=True`. Django stores empty strings, not NULL, for text fields.
- **Choosing the wrong `on_delete` behavior**: Using `CASCADE` means deleting a parent deletes all children. Use `PROTECT` if you want to prevent accidental deletion of related data.

## Best Practices

- **Always define `__str__`** on your models so they display meaningfully in the admin and shell.
- **Use `blank=True` for optional text fields** and `null=True` only for non-text fields (dates, numbers, foreign keys).
- **Add `verbose_name` to fields** when the field name alone is not descriptive enough for the admin interface.

## Summary

- Models are Python classes that map to **database tables** via Django's ORM
- Each model attribute represents a **database column** with a specific field type
- Common field types include `CharField`, `TextField`, `IntegerField`, `DateTimeField`, and `BooleanField`
- `ForeignKey` creates **many-to-one relationships** between models
- Field options like `default`, `blank`, `null`, and `max_length` control validation and storage behavior

## Code Examples

**Defining Django models with fields and a ForeignKey relationship**

```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```


## Resources

- [Django Models](https://docs.djangoproject.com/en/6.0/topics/db/models/) — Official guide to Django models

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*