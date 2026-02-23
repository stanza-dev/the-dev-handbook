---
source_course: "django-foundations"
source_lesson: "django-foundations-admin-setup"
---

# Setting Up the Admin

Django's admin interface is one of its most powerful features. It automatically generates a full-featured administrative interface for your models.

## Creating a Superuser

First, create an admin account:

```bash
python manage.py createsuperuser
```

You'll be prompted for:
- Username
- Email address
- Password (entered twice)

```
Username: admin
Email address: admin@example.com
Password: ********
Password (again): ********
Superuser created successfully.
```

## Accessing the Admin

Start the development server:

```bash
python manage.py runserver
```

Visit **http://127.0.0.1:8000/admin/** and log in with your superuser credentials.

## Registering Models

By default, only Django's built-in models appear. Register your own models:

```python
# polls/admin.py
from django.contrib import admin
from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```

Now `Question` and `Choice` appear in the admin!

## Understanding the Admin Interface

The admin provides:

- **List view**: See all objects with sorting and filtering
- **Add view**: Create new objects with validated forms
- **Change view**: Edit existing objects
- **Delete view**: Remove objects with confirmation
- **History**: Track who changed what and when

## How It Works

Django's admin uses your model definitions to:
1. Generate form fields from model fields
2. Apply validation from model constraints
3. Handle relationships (ForeignKey, ManyToMany)
4. Provide sensible defaults for display

## The __str__ Method

Improve how objects display in admin:

```python
# polls/models.py
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text  # Shows in admin lists


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
```

Without `__str__`, you'd see unhelpful names like "Question object (1)".

## Common Pitfalls

- **Forgetting to create a superuser**: You cannot access the admin without a superuser account. Run `python manage.py createsuperuser` first.
- **Not defining `__str__` on models**: Without `__str__`, the admin shows unhelpful names like "Question object (1)" instead of meaningful labels.
- **Registering a model twice**: Calling `admin.site.register(Model)` twice for the same model raises an `AlreadyRegistered` error.

## Best Practices

- **Always define `__str__`** on every model for clear representation in the admin and shell.
- **Use the `@admin.register` decorator** instead of `admin.site.register()` for cleaner syntax.
- **Run migrations before creating a superuser**: The auth tables must exist in the database first.

## Summary

- Django's admin interface **auto-generates CRUD pages** from your model definitions
- Create a superuser with `python manage.py createsuperuser` to access `/admin/`
- Register models in `admin.py` with `admin.site.register(Model)` to make them appear
- Define `__str__()` on models for human-readable display in the admin
- The admin provides list views, add/change forms, delete confirmation, and change history

## Code Examples

**Registering models with the Django admin and adding __str__ for readable display**

```python
# polls/admin.py
from django.contrib import admin
from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)

# polls/models.py
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text
```


## Resources

- [The Django Admin Site](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/) — Official reference for Django's admin interface

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*