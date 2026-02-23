---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-model-formsets"
---

## Introduction

Model formsets combine formsets with the ORM, editing multiple model instances with a single save() call.

## Key Concepts

- **modelformset_factory()**: Creates model-backed formset.
- **queryset**: Filters existing objects for editing.
- **formset.save()**: Persists creates, updates, deletes.
- **inlineformset_factory()**: For ForeignKey parent-child editing.
- **prefix**: Namespaces field names.

## Real World Context

A blog admin edits all articles at once. An order system manages line items on one page. Model formsets make bulk editing straightforward.

## Deep Dive

# Model Formsets

Model formsets work with querysets, making it easy to edit multiple model instances.

## Basic Model Formset

```python
from django.forms import modelformset_factory
from .models import Book


BookFormSet = modelformset_factory(
    Book,
    fields=['title', 'author', 'year'],
    extra=2,
    can_delete=True,
)
```

## Using Model Formsets

```python
def manage_books(request):
    if request.method == 'POST':
        formset = BookFormSet(request.POST)
        if formset.is_valid():
            formset.save()  # Handles create, update, delete!
            return redirect('book_list')
    else:
        # Display formset with existing books
        formset = BookFormSet(queryset=Book.objects.filter(is_published=True))
    
    return render(request, 'books/manage.html', {'formset': formset})
```

## Filtering the Queryset

```python
def manage_user_books(request):
    # Only show books belonging to current user
    formset = BookFormSet(
        request.POST or None,
        queryset=Book.objects.filter(owner=request.user)
    )
    
    if request.method == 'POST' and formset.is_valid():
        instances = formset.save(commit=False)
        for instance in instances:
            instance.owner = request.user  # Set owner
            instance.save()
        formset.save_m2m()  # Save many-to-many relations
        return redirect('book_list')
```

## Inline Formsets

Edit related objects (like editing chapters within a book):

```python
from django.forms import inlineformset_factory
from .models import Book, Chapter


ChapterFormSet = inlineformset_factory(
    Book,          # Parent model
    Chapter,       # Child model
    fields=['title', 'content', 'order'],
    extra=2,
    can_delete=True,
)
```

```python
def edit_book_chapters(request, book_id):
    book = get_object_or_404(Book, pk=book_id)
    
    if request.method == 'POST':
        formset = ChapterFormSet(request.POST, instance=book)
        if formset.is_valid():
            formset.save()
            return redirect('book_detail', pk=book_id)
    else:
        formset = ChapterFormSet(instance=book)
    
    return render(request, 'books/chapters.html', {
        'book': book,
        'formset': formset
    })
```

## Multiple Formsets on One Page

```python
def edit_book(request, book_id):
    book = get_object_or_404(Book, pk=book_id)
    
    ChapterFormSet = inlineformset_factory(Book, Chapter, fields=['title'])
    ImageFormSet = inlineformset_factory(Book, Image, fields=['image', 'caption'])
    
    if request.method == 'POST':
        chapter_formset = ChapterFormSet(request.POST, instance=book, prefix='chapters')
        image_formset = ImageFormSet(request.POST, request.FILES, instance=book, prefix='images')
        
        if chapter_formset.is_valid() and image_formset.is_valid():
            chapter_formset.save()
            image_formset.save()
            return redirect('book_detail', pk=book_id)
    else:
        chapter_formset = ChapterFormSet(instance=book, prefix='chapters')
        image_formset = ImageFormSet(instance=book, prefix='images')
    
    return render(request, 'books/edit.html', {
        'book': book,
        'chapter_formset': chapter_formset,
        'image_formset': image_formset,
    })
```

```html
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    
    <h2>Chapters</h2>
    {{ chapter_formset.management_form }}
    {% for form in chapter_formset %}
        {{ form.as_p }}
    {% endfor %}
    
    <h2>Images</h2>
    {{ image_formset.management_form }}
    {% for form in image_formset %}
        {{ form.as_p }}
    {% endfor %}
    
    <button type="submit">Save</button>
</form>
```

## Custom Model Formset

```python
from django.forms import BaseModelFormSet


class BaseBookFormSet(BaseModelFormSet):
    def clean(self):
        super().clean()
        
        # Ensure at least one book is marked as featured
        featured_count = 0
        for form in self.forms:
            if not form.cleaned_data.get('DELETE', False):
                if form.cleaned_data.get('is_featured'):
                    featured_count += 1
        
        if featured_count == 0:
            raise forms.ValidationError('At least one book must be featured.')


BookFormSet = modelformset_factory(
    Book,
    fields=['title', 'is_featured'],
    formset=BaseBookFormSet,
)
```

## Common Pitfalls

1. **Unfiltered queryset** -- Loads ALL objects. Always filter.
2. **Missing save_m2m()** -- Required after commit=False.
3. **No prefix** -- Multiple formsets corrupt each other.

## Best Practices

1. **Scope querysets to user** -- Prevent unauthorized editing.
2. **Use inlineformset_factory** -- For parent-child relationships.
3. **commit=False for extra fields** -- Set modified_by etc.

## Summary

- modelformset_factory creates ORM-backed formsets.
- formset.save() handles create/update/delete.
- Always filter querysets.
- Use inlineformset_factory for ForeignKey editing.
- Call save_m2m() after commit=False.

## Resources

- [Model Formsets](https://docs.djangoproject.com/en/6.0/topics/forms/modelforms/#model-formsets) — Model formsets documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*