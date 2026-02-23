---
source_course: "django-foundations"
source_lesson: "django-foundations-admin-actions"
---

# Admin Actions

## Introduction

Admin actions let you apply bulk operations to multiple objects selected in the list view. Instead of editing records one by one, you can select dozens of items and apply an action with a single click.

## Key Concepts

- **Admin Action**: A function that operates on a queryset of selected objects from the admin list view.
- **Built-in Action**: Django includes a default "Delete selected" action.
- **Custom Action**: A function or method you define to perform batch operations.

## Real World Context

Content management systems frequently need bulk operations: publishing multiple draft articles, archiving old records, sending notifications to selected users, or exporting data. Admin actions make these operations available with no custom views or JavaScript.

## Deep Dive

### The Default Delete Action

Django's admin includes a "Delete selected" action by default. It appears as a dropdown above the list view.

### Creating Custom Actions

Define actions as standalone functions:

```python
# polls/admin.py
from django.contrib import admin
from .models import Question


@admin.action(description="Mark selected questions as published")
def make_published(modeladmin, request, queryset):
    queryset.update(is_active=True)


@admin.action(description="Mark selected questions as draft")
def make_draft(modeladmin, request, queryset):
    queryset.update(is_active=False)


@admin.register(Question)
class QuestionAdmin(admin.ModelAdmin):
    list_display = ['question_text', 'pub_date', 'is_active']
    actions = [make_published, make_draft]
```

### Actions as ModelAdmin Methods

Define actions directly on the ModelAdmin class:

```python
@admin.register(Question)
class QuestionAdmin(admin.ModelAdmin):
    list_display = ['question_text', 'pub_date', 'is_active']
    actions = ['make_published', 'reset_votes']

    @admin.action(description="Mark selected questions as published")
    def make_published(self, request, queryset):
        updated = queryset.update(is_active=True)
        self.message_user(
            request,
            f"{updated} questions were marked as published."
        )

    @admin.action(description="Reset votes on all choices")
    def reset_votes(self, request, queryset):
        for question in queryset:
            question.choice_set.update(votes=0)
        self.message_user(request, "Votes have been reset.")
```

### Action Parameters

Every action receives three arguments:

```python
def my_action(modeladmin, request, queryset):
    # modeladmin: The ModelAdmin instance
    # request: The current HttpRequest
    # queryset: QuerySet of selected objects
    pass
```

### User Feedback with message_user

After an action completes, you should inform the user about what happened. The `message_user()` method displays a success notification in the admin.

```python
from django.contrib import messages

@admin.action(description="Archive selected articles")
def archive_articles(self, request, queryset):
    count = queryset.update(status='archived')
    self.message_user(
        request,
        f"Successfully archived {count} articles.",
        messages.SUCCESS
    )
```

### Disabling the Default Delete Action

If you want to prevent bulk deletion from the admin, you can override `get_actions()` to remove the built-in "Delete selected" action.

```python
@admin.register(Question)
class QuestionAdmin(admin.ModelAdmin):
    actions = ['make_published']

    def get_actions(self, request):
        actions = super().get_actions(request)
        if 'delete_selected' in actions:
            del actions['delete_selected']
        return actions
```

### Actions with Intermediate Pages

For actions that need confirmation or extra input:

```python
from django.http import HttpResponse
import csv

@admin.action(description="Export selected as CSV")
def export_as_csv(modeladmin, request, queryset):
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="export.csv"'
    writer = csv.writer(response)
    writer.writerow(['Question', 'Published Date', 'Active'])
    for obj in queryset:
        writer.writerow([obj.question_text, obj.pub_date, obj.is_active])
    return response
```

## Common Pitfalls

- **Forgetting `@admin.action(description=...)`**: Without the `description`, the action dropdown shows the raw function name (e.g., `make_published`) instead of a human-readable label.
- **Not using `queryset.update()` for bulk operations**: Looping through objects and calling `.save()` on each one is much slower than using `queryset.update()` for simple field changes.
- **Not providing user feedback**: Always call `self.message_user()` to confirm the action completed successfully.

## Best Practices

- **Use `queryset.update()`** for simple field changes to avoid N+1 database queries.
- **Add confirmation for destructive actions**: Return an intermediate template for actions that delete or irreversibly modify data.
- **Keep action names descriptive**: Use clear descriptions like "Mark as published" rather than "Update status".

## Summary

- Admin actions enable **bulk operations** on selected objects from the list view
- Define actions as standalone functions with `@admin.action(description=...)` or as ModelAdmin methods
- Every action receives `modeladmin`, `request`, and `queryset` as arguments
- Use `self.message_user()` to provide feedback after the action completes
- Use `queryset.update()` for efficient bulk field updates

## Code Examples

**Custom admin action defined as a ModelAdmin method with user feedback**

```python
@admin.register(Question)
class QuestionAdmin(admin.ModelAdmin):
    list_display = ['question_text', 'pub_date', 'is_active']
    actions = ['make_published']

    @admin.action(description="Mark selected questions as published")
    def make_published(self, request, queryset):
        updated = queryset.update(is_active=True)
        self.message_user(request, f"{updated} questions published.")
```


## Resources

- [Admin Actions](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/actions/) — Official guide to Django admin actions

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*