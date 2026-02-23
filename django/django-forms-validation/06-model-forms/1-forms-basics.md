---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-model-forms-basics"
---

## Introduction

ModelForms generate form fields from model definitions, keeping forms and models in sync automatically.

## Key Concepts

- **ModelForm**: Reads field definitions from a model Meta class.
- **fields/exclude**: Control which model fields appear. Prefer explicit fields.
- **form.save()**: Creates or updates a model instance.
- **commit=False**: Returns instance without saving for modification.
- **save_m2m()**: Saves ManyToMany after commit=False.

## Real World Context

A CMS ArticleForm with fields=[title, body, status] sets author via commit=False. When a model field changes, the form automatically picks up the change.

## Deep Dive

# ModelForm Basics

ModelForms automatically generate form fields from your model definitions, reducing code duplication and keeping forms in sync with models.

## Basic ModelForm

```python
from django import forms
from .models import Article


class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'body', 'category', 'tags']
```

This automatically creates:
- CharField for title (max_length from model)
- Textarea for body (TextField)
- Select for category (ForeignKey)
- Multiple select for tags (ManyToMany)

## Field Selection

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        
        # Explicit list (recommended)
        fields = ['title', 'body', 'status']
        
        # All fields (avoid - security risk)
        # fields = '__all__'
        
        # Exclude specific fields
        # exclude = ['created_at', 'updated_at', 'author']
```

⚠️ **Never use `fields = '__all__'` or `exclude`** with sensitive fields. Always use explicit field lists.

## Customizing Fields

```python
class ArticleForm(forms.ModelForm):
    # Override or extend fields
    title = forms.CharField(
        max_length=200,
        help_text='Enter an engaging title',
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Article title'
        })
    )
    
    # Add extra field not on model
    notify_subscribers = forms.BooleanField(required=False)
    
    class Meta:
        model = Article
        fields = ['title', 'body', 'status']
        
        # Widget customization
        widgets = {
            'body': forms.Textarea(attrs={
                'rows': 10,
                'class': 'form-control'
            }),
            'status': forms.RadioSelect(),
        }
        
        # Labels
        labels = {
            'body': 'Article Content',
        }
        
        # Help text
        help_texts = {
            'status': 'Choose publication status',
        }
        
        # Error messages
        error_messages = {
            'title': {
                'required': 'Please provide a title',
                'max_length': 'Title is too long',
            },
        }
```

## Saving ModelForms

```python
def create_article(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            # Save creates and saves the model instance
            article = form.save()
            return redirect('article_detail', pk=article.pk)
    else:
        form = ArticleForm()
    
    return render(request, 'create.html', {'form': form})
```

## commit=False Pattern

Used when you need to modify the instance before saving:

```python
def create_article(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            # Don't save to database yet
            article = form.save(commit=False)
            
            # Add data not from form
            article.author = request.user
            article.ip_address = request.META.get('REMOTE_ADDR')
            
            # Now save to database
            article.save()
            
            # For ManyToMany fields with commit=False:
            form.save_m2m()  # Save tags, categories, etc.
            
            return redirect('article_detail', pk=article.pk)
```

## Updating Existing Objects

```python
def edit_article(request, pk):
    article = get_object_or_404(Article, pk=pk)
    
    if request.method == 'POST':
        # Pass instance to update existing object
        form = ArticleForm(request.POST, instance=article)
        if form.is_valid():
            form.save()
            return redirect('article_detail', pk=pk)
    else:
        # Pre-populate form with existing data
        form = ArticleForm(instance=article)
    
    return render(request, 'edit.html', {'form': form, 'article': article})
```

## Initial Values

```python
def create_article(request):
    # Provide initial values
    form = ArticleForm(initial={
        'status': 'draft',
        'category': Category.objects.get(slug='general'),
    })
    
    return render(request, 'create.html', {'form': form})
```

## Common Pitfalls

1. **Using fields=__all__** -- Exposes sensitive fields. Always use explicit list.
2. **Forgetting save_m2m()** -- M2M needs a PK first.
3. **No instance= for updates** -- Creates new record instead of updating.

## Best Practices

1. **Override widgets in Meta** -- Preserves model validators.
2. **Use commit=False for non-form data** -- author, ip_address etc.
3. **Customize labels in Meta** -- Keep model definitions clean.

## Summary

- ModelForm generates fields from model definitions.
- Always use explicit fields list in Meta.
- Pass instance= for updates.
- Use commit=False then save_m2m() for M2M.
- Override widgets/labels in Meta.

## Code Examples

**ModelForm automatically creates form fields from model definitions -- pass instance= to edit an existing object**

```python
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'body', 'category', 'tags']

# Create
form = ArticleForm(request.POST)
if form.is_valid():
    article = form.save()  # creates and saves to database

# Update existing
article = Article.objects.get(pk=1)
form = ArticleForm(request.POST, instance=article)
if form.is_valid():
    form.save()  # updates the existing article
```


## Resources

- [ModelForm](https://docs.djangoproject.com/en/6.0/topics/forms/modelforms/) — Official ModelForm documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*