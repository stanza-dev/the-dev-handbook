---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-formset-basics"
---

# Formset Basics

Formsets manage multiple instances of a form on a single page—perfect for editing multiple items at once.

## Creating a Formset

```python
from django import forms
from django.forms import formset_factory


class BookForm(forms.Form):
    title = forms.CharField(max_length=200)
    author = forms.CharField(max_length=100)
    year = forms.IntegerField(min_value=1000, max_value=2100)


# Create formset class
BookFormSet = formset_factory(BookForm, extra=3)
```

## Using Formsets in Views

```python
def manage_books(request):
    if request.method == 'POST':
        formset = BookFormSet(request.POST)
        if formset.is_valid():
            for form in formset:
                if form.cleaned_data:  # Skip empty forms
                    title = form.cleaned_data['title']
                    author = form.cleaned_data['author']
                    Book.objects.create(title=title, author=author)
            return redirect('book_list')
    else:
        formset = BookFormSet()
    
    return render(request, 'books/manage.html', {'formset': formset})
```

## Rendering Formsets

```html
<!-- templates/books/manage.html -->
<form method="post">
    {% csrf_token %}
    
    <!-- Management form (required!) -->
    {{ formset.management_form }}
    
    <table>
        <thead>
            <tr>
                <th>Title</th>
                <th>Author</th>
                <th>Year</th>
            </tr>
        </thead>
        <tbody>
        {% for form in formset %}
            <tr>
                <td>{{ form.title }}</td>
                <td>{{ form.author }}</td>
                <td>{{ form.year }}</td>
                {% if form.errors %}
                    <td>{{ form.errors }}</td>
                {% endif %}
            </tr>
        {% endfor %}
        </tbody>
    </table>
    
    <button type="submit">Save All</button>
</form>
```

## Formset Options

```python
BookFormSet = formset_factory(
    BookForm,
    extra=3,              # Number of empty forms
    max_num=10,           # Maximum forms allowed
    min_num=1,            # Minimum forms required
    validate_max=True,    # Validate max_num
    validate_min=True,    # Validate min_num
    can_delete=True,      # Add delete checkbox
    can_order=True,       # Add order field
)
```

## Handling Deletions

```python
BookFormSet = formset_factory(BookForm, can_delete=True)

def manage_books(request):
    if request.method == 'POST':
        formset = BookFormSet(request.POST)
        if formset.is_valid():
            for form in formset:
                if form.cleaned_data.get('DELETE'):
                    # This form is marked for deletion
                    continue
                if form.cleaned_data:
                    # Process form...
                    pass
```

```html
{% for form in formset %}
<tr>
    <td>{{ form.title }}</td>
    <td>{{ form.author }}</td>
    <td>{{ form.DELETE }} Delete</td>  <!-- Checkbox -->
</tr>
{% endfor %}
```

## Formset Validation

```python
from django.forms import BaseFormSet


class BaseBookFormSet(BaseFormSet):
    def clean(self):
        """Cross-form validation."""
        if any(self.errors):
            return  # Don't validate if individual forms have errors
        
        titles = []
        for form in self.forms:
            if form.cleaned_data and not form.cleaned_data.get('DELETE'):
                title = form.cleaned_data.get('title')
                if title in titles:
                    raise forms.ValidationError('Duplicate titles not allowed.')
                titles.append(title)
        
        if len(titles) < 2:
            raise forms.ValidationError('Please add at least 2 books.')


BookFormSet = formset_factory(
    BookForm,
    formset=BaseBookFormSet,
    extra=3
)
```

## Initial Data

```python
def manage_books(request):
    initial_data = [
        {'title': 'Django for Beginners', 'author': 'William Vincent', 'year': 2020},
        {'title': 'Two Scoops of Django', 'author': 'Daniel Feldroy', 'year': 2021},
    ]
    
    formset = BookFormSet(initial=initial_data)
    return render(request, 'books/manage.html', {'formset': formset})
```

## Resources

- [Formsets](https://docs.djangoproject.com/en/6.0/topics/forms/formsets/) — Official formsets documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*