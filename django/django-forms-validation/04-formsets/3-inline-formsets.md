---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-inline-formsets"
---

# Inline Formsets

## Introduction

Inline formsets edit child objects through a parent relationship, like editing chapters for a book.

## Key Concepts

- **inlineformset_factory()**: Creates formset for ForeignKey child objects.
- **instance**: Parent model object for both GET and POST.
- **can_delete**: Delete checkbox on each child form.

## Real World Context

A course editor where each course has multiple lessons. Inline formset edits all lessons on one page with add/edit/delete.

## Deep Dive

### Creating an Inline Formset

```python
from django.forms import inlineformset_factory

ChapterFormSet = inlineformset_factory(
    Book,      # Parent model
    Chapter,   # Child model
    fields=['title', 'content'],
    extra=2,
    can_delete=True,
)
```

### Using in Views

```python
def edit_book(request, pk):
    book = get_object_or_404(Book, pk=pk)
    
    if request.method == 'POST':
        formset = ChapterFormSet(request.POST, instance=book)
        if formset.is_valid():
            formset.save()
            return redirect('book_detail', pk=pk)
    else:
        formset = ChapterFormSet(instance=book)
    
    return render(request, 'books/edit.html', {'book': book, 'formset': formset})
```

## Common Pitfalls

1. **Passing child not parent instance** -- Causes ValueError.
2. **Missing instance on POST** -- Cannot associate new children with parent.

## Best Practices

1. **Use get_object_or_404 for parent** -- Prevents orphaned children.
2. **Set extra=1 for production** -- Less clutter than default extra=3.

## Summary

Inline formsets link child forms to a parent instance. Pass instance= to both GET and POST handling.

## Resources

- [Inline Formsets](https://docs.djangoproject.com/en/6.0/topics/forms/modelforms/#inline-formsets) — Inline formsets

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*