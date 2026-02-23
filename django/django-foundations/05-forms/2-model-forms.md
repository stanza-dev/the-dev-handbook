---
source_course: "django-foundations"
source_lesson: "django-foundations-model-forms"
---

# Model Forms

ModelForms automatically generate form fields from your models, reducing duplication and keeping your forms in sync with your data structure.

## Creating a ModelForm

A ModelForm starts with the model it mirrors. Define the model first, then create a form class that references it through an inner `Meta` class.

```python
# polls/models.py
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
    is_active = models.BooleanField(default=True)
```

```python
# polls/forms.py
from django import forms
from .models import Question


class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question
        fields = ['question_text', 'pub_date', 'is_active']
```

## Meta Class Options

The `Meta` class on a ModelForm gives you fine-grained control over which fields to include, how they appear, and what error messages to display.

```python
class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question
        
        # Include specific fields
        fields = ['question_text', 'pub_date']
        
        # Or exclude specific fields
        # exclude = ['is_active']
        
        # Or include all fields (not recommended)
        # fields = '__all__'
        
        # Custom labels
        labels = {
            'question_text': 'Your Question',
            'pub_date': 'Publication Date',
        }
        
        # Custom help text
        help_texts = {
            'question_text': 'Enter your question here.',
        }
        
        # Custom widgets
        widgets = {
            'question_text': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Enter your question'
            }),
            'pub_date': forms.DateTimeInput(attrs={
                'type': 'datetime-local'
            }),
        }
        
        # Custom error messages
        error_messages = {
            'question_text': {
                'required': 'Please enter a question.',
                'max_length': 'Question is too long.',
            },
        }
```

## Saving ModelForms

ModelForms can save directly to the database:

```python
# polls/views.py
def create_question(request):
    if request.method == 'POST':
        form = QuestionForm(request.POST)
        if form.is_valid():
            question = form.save()  # Creates and saves the object
            return redirect('polls:detail', pk=question.pk)
    else:
        form = QuestionForm()
    
    return render(request, 'polls/create.html', {'form': form})
```

### Delayed Save

Modify the object before saving:

```python
if form.is_valid():
    question = form.save(commit=False)  # Don't save yet
    question.author = request.user       # Add extra data
    question.save()                       # Now save
```

## Editing Existing Objects

Pass an instance to edit:

```python
def edit_question(request, pk):
    question = get_object_or_404(Question, pk=pk)
    
    if request.method == 'POST':
        form = QuestionForm(request.POST, instance=question)
        if form.is_valid():
            form.save()
            return redirect('polls:detail', pk=pk)
    else:
        form = QuestionForm(instance=question)  # Pre-filled form
    
    return render(request, 'polls/edit.html', {'form': form})
```

## Adding Custom Validation

ModelForms support the same validation hooks as regular forms. Use `clean_fieldname()` for single-field validation and `clean()` for cross-field rules.

```python
class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question
        fields = ['question_text', 'pub_date']
    
    def clean_question_text(self):
        """Validate the question_text field."""
        text = self.cleaned_data['question_text']
        if not text.endswith('?'):
            raise forms.ValidationError('Questions must end with a question mark.')
        return text
    
    def clean(self):
        """Cross-field validation."""
        cleaned_data = super().clean()
        pub_date = cleaned_data.get('pub_date')
        
        if pub_date and pub_date > timezone.now():
            raise forms.ValidationError('Publication date cannot be in the future.')
        
        return cleaned_data
```

## Common Pitfalls

- **Using `fields = '__all__'` in ModelForm**: This exposes every model field, including sensitive ones. Always list fields explicitly for security.
- **Forgetting `commit=False` when setting extra fields**: If your view needs to add data (like `author = request.user`) before saving, use `form.save(commit=False)` first.
- **Not calling `form.save_m2m()`**: When using `commit=False` on a ModelForm with ManyToManyField, you must call `form.save_m2m()` after saving the instance.

## Best Practices

- **List fields explicitly** in the `Meta.fields` attribute rather than using `'__all__'` or `exclude`.
- **Use the `widgets` dictionary** in Meta to customize HTML attributes like CSS classes and input types.
- **Add custom validation** with `clean_fieldname()` methods for field-level validation or `clean()` for cross-field validation.

## Summary

- ModelForms **auto-generate form fields** from Django model definitions, reducing duplication
- Configure fields, labels, widgets, and error messages in the inner `Meta` class
- Use `form.save()` to create/update database records directly from form data
- Use `form.save(commit=False)` to modify the instance before saving
- Add custom validation with `clean_fieldname()` and `clean()` methods

## Code Examples

**A ModelForm that auto-generates form fields from a Django model with custom widgets**

```python
from django import forms
from .models import Question

class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question
        fields = ['question_text', 'pub_date']
        widgets = {
            'question_text': forms.TextInput(attrs={'class': 'form-control'}),
            'pub_date': forms.DateTimeInput(attrs={'type': 'datetime-local'}),
        }
```


## Resources

- [Creating Forms from Models](https://docs.djangoproject.com/en/6.0/topics/forms/modelforms/) — Official guide to ModelForms

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*