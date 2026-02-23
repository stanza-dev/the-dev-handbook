---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-inline-types"
---

# Inline Model Admin

## Introduction

Inlines let you edit related models on the same page as the parent model, eliminating the need to navigate between separate admin pages. In this lesson, you will learn the differences between TabularInline and StackedInline, and how to configure them for various use cases.

## Key Concepts

**TabularInline**: Displays each related object as a row in a compact table.

**StackedInline**: Displays each related object vertically with full-width fields.

**extra**: Number of empty forms shown for adding new related objects.

**GenericTabularInline**: Inline for models using GenericForeignKey relationships.

**show_change_link**: Adds a link to the full change form of the inline object.

## Real World Context

An online bookstore admin needs to manage books and their chapters. Without inlines, an editor would add a book, save it, navigate to chapters, add each chapter with a dropdown to find the parent book, and repeat. With a ChapterInline on BookAdmin, the editor adds all chapters directly on the book's page in a single form submission.

## Deep Dive

Inlines let you edit related models on the same page as the parent model.

### TabularInline vs StackedInline

```python
from django.contrib import admin
from .models import Author, Book, Chapter


class BookInline(admin.TabularInline):
    """Compact table format."""
    model = Book
    extra = 1  # Number of empty forms


class ChapterInline(admin.StackedInline):
    """Each item stacked vertically."""
    model = Chapter
    extra = 0


@admin.register(Author)
class AuthorAdmin(admin.ModelAdmin):
    inlines = [BookInline]


@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    inlines = [ChapterInline]
```

### Inline Configuration

```python
class ChapterInline(admin.TabularInline):
    model = Chapter
    
    # Number of empty forms
    extra = 1
    
    # Minimum forms (even if no data)
    min_num = 1
    
    # Maximum forms
    max_num = 20
    
    # Fields to display
    fields = ['title', 'content', 'order']
    
    # Read-only fields
    readonly_fields = ['created_at']
    
    # Order inlines
    ordering = ['order']
    
    # Allow deletion
    can_delete = True
    
    # Show change link
    show_change_link = True
    
    # Custom verbose names
    verbose_name = 'Chapter'
    verbose_name_plural = 'Chapters'
```

### Filtering Inline Queryset

```python
class PublishedChapterInline(admin.TabularInline):
    model = Chapter
    verbose_name = 'Published Chapter'
    verbose_name_plural = 'Published Chapters'
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        return qs.filter(is_published=True)


class DraftChapterInline(admin.TabularInline):
    model = Chapter
    verbose_name = 'Draft Chapter'
    verbose_name_plural = 'Draft Chapters'
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        return qs.filter(is_published=False)


@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    inlines = [PublishedChapterInline, DraftChapterInline]
```

### Read-Only Inline

```python
class OrderItemInline(admin.TabularInline):
    model = OrderItem
    extra = 0
    can_delete = False
    
    # Make all fields read-only
    readonly_fields = ['product', 'quantity', 'price']
    
    def has_add_permission(self, request, obj=None):
        return False
    
    def has_change_permission(self, request, obj=None):
        return False
```

### GenericInline for Generic Relations

```python
from django.contrib.contenttypes.admin import GenericTabularInline
from .models import Comment


class CommentInline(GenericTabularInline):
    model = Comment
    extra = 0
    ct_field = 'content_type'  # ContentType field name
    ct_fk_field = 'object_id'   # Object ID field name


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    inlines = [CommentInline]


@admin.register(Photo)
class PhotoAdmin(admin.ModelAdmin):
    inlines = [CommentInline]  # Same inline works for any model
```

### Inline Formsets

```python
from django.forms import BaseInlineFormSet


class RequiredChapterFormSet(BaseInlineFormSet):
    """Require at least one chapter."""
    
    def clean(self):
        super().clean()
        
        # Count non-deleted forms with data
        count = 0
        for form in self.forms:
            if form.cleaned_data and not form.cleaned_data.get('DELETE'):
                count += 1
        
        if count < 1:
            raise ValidationError('At least one chapter is required.')


class ChapterInline(admin.TabularInline):
    model = Chapter
    formset = RequiredChapterFormSet
    extra = 1
```

## Common Pitfalls

1. **Setting extra too high**: The default extra=3 creates 3 empty forms per inline. For models with many inlines or complex forms, this can make the page slow and cluttered. Set extra=0 or extra=1 for most use cases.

2. **Forgetting show_change_link=True**: Without this, users cannot navigate to the inline object's own change form for detailed editing. Enable it when inlines have many fields you cannot display in the compact view.

3. **Not using min_num/max_num for validation**: If a parent requires at least one child (e.g., an order needs at least one line item), set min_num=1 to enforce this at the form level.

## Best Practices

1. **Choose TabularInline for simple models**: When inline objects have 3-5 fields, tabular layout is more scannable and space-efficient than stacked.

2. **Use ordering on inlines**: Set `ordering = ['position']` or similar to ensure inline objects appear in a predictable order.

3. **Override get_queryset() for large datasets**: Filter the inline queryset to avoid loading thousands of related objects on every page load.

## Summary

- TabularInline shows related objects as compact table rows; StackedInline shows them vertically.
- Configure extra, min_num, max_num to control the number of forms.
- GenericTabularInline works with GenericForeignKey relations.
- Use show_change_link, ordering, and get_queryset() for better usability.
- Custom formsets enable cross-inline validation.

## Code Examples

**TabularInline for editing related Comment objects within the Article admin page**

```python
class CommentInline(admin.TabularInline):
    model = Comment
    extra = 1
    readonly_fields = ['created_at']

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    inlines = [CommentInline]
```


## Resources

- [InlineModelAdmin](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#inlinemodeladmin-objects) — Complete inline admin reference

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*