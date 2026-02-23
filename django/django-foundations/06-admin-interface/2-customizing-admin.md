---
source_course: "django-foundations"
source_lesson: "django-foundations-customizing-admin"
---

# Customizing the Admin

While the default admin is useful, customizing it makes it much more powerful for your specific needs.

## Using ModelAdmin

Create a `ModelAdmin` class to customize how a model appears:

```python
# polls/admin.py
from django.contrib import admin
from .models import Question, Choice


class QuestionAdmin(admin.ModelAdmin):
    # Fields to display in list view
    list_display = ['question_text', 'pub_date', 'was_published_recently']
    
    # Add filters to the sidebar
    list_filter = ['pub_date']
    
    # Add a search box
    search_fields = ['question_text']
    
    # Default ordering
    ordering = ['-pub_date']
    
    # Add date hierarchy navigation
    date_hierarchy = 'pub_date'


admin.site.register(Question, QuestionAdmin)
admin.site.register(Choice)
```

## Customizing the Change Form

Control how the edit form looks:

```python
class QuestionAdmin(admin.ModelAdmin):
    # Group fields into fieldsets
    fieldsets = [
        (None, {'fields': ['question_text']}),
        ('Date Information', {
            'fields': ['pub_date'],
            'classes': ['collapse'],  # Collapsible section
        }),
    ]
    
    # Or simple field ordering
    # fields = ['pub_date', 'question_text']
    
    # Read-only fields
    readonly_fields = ['pub_date']
```

## Inline Editing

Edit related objects on the same page:

```python
class ChoiceInline(admin.TabularInline):  # Or admin.StackedInline
    model = Choice
    extra = 3  # Show 3 empty forms


class QuestionAdmin(admin.ModelAdmin):
    list_display = ['question_text', 'pub_date']
    inlines = [ChoiceInline]  # Edit choices on question page


admin.site.register(Question, QuestionAdmin)
# Don't need to register Choice separately anymore
```

## List View Customization

The list view supports several options for controlling how data is displayed, which fields are editable inline, and how many items appear per page.

```python
class QuestionAdmin(admin.ModelAdmin):
    # Columns to display
    list_display = ['question_text', 'pub_date', 'choice_count']
    
    # Make fields editable in list view
    list_editable = ['pub_date']
    
    # Clickable fields (link to change page)
    list_display_links = ['question_text']
    
    # Items per page
    list_per_page = 25
    
    # Custom method for display
    @admin.display(description='Number of Choices')
    def choice_count(self, obj):
        return obj.choice_set.count()
```

## Using the Decorator Syntax

A cleaner way to register:

```python
from django.contrib import admin
from .models import Question


@admin.register(Question)
class QuestionAdmin(admin.ModelAdmin):
    list_display = ['question_text', 'pub_date']
    search_fields = ['question_text']
```

## Admin Site Customization

Customize the admin site itself:

```python
# mysite/admin.py or polls/admin.py
admin.site.site_header = 'My Site Administration'
admin.site.site_title = 'My Site Admin'
admin.site.index_title = 'Welcome to the Admin Panel'
```

## Common Pitfalls

- **Putting a non-existent field in `list_display`**: Django raises a `FieldDoesNotExist` error. Double-check field names match your model.
- **Using `list_editable` without `list_display`**: Fields in `list_editable` must also appear in `list_display`, and the first field in `list_display` cannot be editable.
- **Forgetting `extra = 0` on inlines**: By default, Django shows 3 empty inline forms. Set `extra = 0` to show only existing related objects.

## Best Practices

- **Use `@admin.register(Model)` decorator** instead of `admin.site.register()` for cleaner code.
- **Add `search_fields` and `list_filter`** to every ModelAdmin for easy data discovery.
- **Use fieldsets** to organize the change form into logical groups with collapsible sections.

## Summary

- Customize the admin by creating `ModelAdmin` classes with options like `list_display`, `list_filter`, and `search_fields`
- Use `fieldsets` to organize edit forms into collapsible sections
- **Inline editing** (`TabularInline`, `StackedInline`) lets you edit related objects on the same page
- Add computed columns with `@admin.display` decorated methods
- Customize the admin site header and title with `admin.site.site_header`

## Code Examples

**Customizing the admin list view with display columns, filters, search, and computed fields**

```python
@admin.register(Question)
class QuestionAdmin(admin.ModelAdmin):
    list_display = ['question_text', 'pub_date', 'choice_count']
    list_filter = ['pub_date']
    search_fields = ['question_text']
    ordering = ['-pub_date']

    @admin.display(description='Number of Choices')
    def choice_count(self, obj):
        return obj.choice_set.count()
```


## Resources

- [ModelAdmin Options](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#modeladmin-options) — Complete reference for ModelAdmin customization options

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*