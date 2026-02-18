---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-inline-types"
---

# Inline Model Admin

Inlines let you edit related models on the same page as the parent model.

## TabularInline vs StackedInline

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

## Inline Configuration

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

## Filtering Inline Queryset

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

## Read-Only Inline

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

## GenericInline for Generic Relations

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

## Inline Formsets

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

## Resources

- [InlineModelAdmin](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#inlinemodeladmin-objects) — Complete inline admin reference

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*