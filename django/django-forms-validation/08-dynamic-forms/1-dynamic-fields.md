---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-dynamic-fields"
---

## Introduction

Dynamic forms have fields determined at runtime, not class definition time. Survey forms, filters, and permission-based editors need this.

## Key Concepts

- **self.fields in __init__**: Modifiable field dictionary.
- **Dynamic choices**: Set ChoiceField choices in __init__.
- **del self.fields[name]**: Remove fields dynamically.
- **kwargs.pop()**: Extract custom constructor arguments.

## Real World Context

A survey platform where admins define questions through a UI. The view passes questions to __init__ which creates fields dynamically for each question.

## Deep Dive

# Dynamic Form Fields

Sometimes you need forms where the fields change based on user data, model state, or other runtime conditions.

## Adding Fields in __init__

```python
from django import forms


class SurveyForm(forms.Form):
    name = forms.CharField(max_length=100)
    
    def __init__(self, questions, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        # Add a field for each question
        for i, question in enumerate(questions):
            field_name = f'question_{i}'
            self.fields[field_name] = forms.CharField(
                label=question.text,
                required=question.required,
                widget=forms.Textarea if question.long_answer else forms.TextInput
            )
```

```python
# views.py
def survey_view(request, survey_id):
    survey = get_object_or_404(Survey, pk=survey_id)
    questions = survey.questions.all()
    
    if request.method == 'POST':
        form = SurveyForm(questions, request.POST)
        if form.is_valid():
            # Process answers
            for i, question in enumerate(questions):
                answer = form.cleaned_data[f'question_{i}']
                # Save answer...
    else:
        form = SurveyForm(questions)
    
    return render(request, 'survey.html', {'form': form})
```

## Conditional Fields Based on User

```python
class ProfileForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ['bio', 'website', 'location']
    
    def __init__(self, user, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        # Add premium-only fields
        if user.is_premium:
            self.fields['custom_theme'] = forms.ChoiceField(
                choices=Profile.THEME_CHOICES
            )
            self.fields['analytics_enabled'] = forms.BooleanField(
                required=False
            )
        
        # Staff can edit more fields
        if user.is_staff:
            self.fields['verified'] = forms.BooleanField(required=False)
```

## Dynamic Choices

```python
class OrderForm(forms.Form):
    product = forms.ChoiceField(choices=[])
    quantity = forms.IntegerField(min_value=1)
    
    def __init__(self, category, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        # Set choices based on category
        products = Product.objects.filter(
            category=category,
            in_stock=True
        )
        self.fields['product'].choices = [
            (p.id, f'{p.name} - ${p.price}')
            for p in products
        ]
```

## Removing Fields Conditionally

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'body', 'status', 'pub_date']
    
    def __init__(self, *args, **kwargs):
        user = kwargs.pop('user', None)
        super().__init__(*args, **kwargs)
        
        # Non-editors can't change status
        if user and not user.has_perm('blog.change_article_status'):
            del self.fields['status']
        
        # Remove pub_date for drafts
        instance = kwargs.get('instance')
        if instance and instance.status == 'draft':
            del self.fields['pub_date']
```

## Form Factory Pattern

```python
def make_survey_form(questions):
    """Create a form class dynamically."""
    fields = {}
    
    for i, q in enumerate(questions):
        if q.type == 'text':
            fields[f'q_{q.id}'] = forms.CharField(label=q.text)
        elif q.type == 'number':
            fields[f'q_{q.id}'] = forms.IntegerField(label=q.text)
        elif q.type == 'choice':
            fields[f'q_{q.id}'] = forms.ChoiceField(
                label=q.text,
                choices=[(c.id, c.text) for c in q.choices.all()]
            )
        elif q.type == 'multiple':
            fields[f'q_{q.id}'] = forms.MultipleChoiceField(
                label=q.text,
                choices=[(c.id, c.text) for c in q.choices.all()],
                widget=forms.CheckboxSelectMultiple
            )
    
    return type('SurveyForm', (forms.Form,), fields)


# Usage
def survey_view(request, survey_id):
    survey = Survey.objects.get(pk=survey_id)
    SurveyForm = make_survey_form(survey.questions.all())
    
    if request.method == 'POST':
        form = SurveyForm(request.POST)
        # ...
```

## Modifying Field Properties

```python
class RegistrationForm(forms.Form):
    username = forms.CharField()
    email = forms.EmailField()
    
    def __init__(self, require_unique_email=True, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        # Modify existing field properties
        if require_unique_email:
            self.fields['email'].help_text = 'Must be unique'
            self.fields['email'].validators.append(unique_email_validator)
        
        # Add CSS classes
        for field in self.fields.values():
            field.widget.attrs['class'] = 'form-control'
```

## Common Pitfalls

1. **Accessing fields before super().__init__()** -- Raises AttributeError.
2. **Modifying class-level fields** -- Call super() first.
3. **Not popping custom kwargs** -- Django __init__ gets unexpected arguments.

## Best Practices

1. **Use kwargs.pop()** -- Cleanly extract custom arguments.
2. **Loop attrs** -- Apply uniform CSS classes.
3. **Document constructor signature** -- Non-standard constructors need docs.

## Summary

- Modify self.fields in __init__ to add/remove/reconfigure.
- Call super().__init__() before accessing fields.
- Pop custom arguments from kwargs.
- Use dynamic choices for database-driven options.
- del self.fields[name] for conditional removal.

## Code Examples

**Dynamic forms add fields in __init__ based on runtime data -- each form instance can have different fields**

```python
from django import forms

class SurveyForm(forms.Form):
    name = forms.CharField(max_length=100)

    def __init__(self, questions, *args, **kwargs):
        super().__init__(*args, **kwargs)
        for i, question in enumerate(questions):
            self.fields[f'question_{i}'] = forms.CharField(
                label=question.text,
                required=question.required,
            )

# Usage: fields are created at runtime based on data
form = SurveyForm(questions=Question.objects.all())
```


## Resources

- [Dynamic Forms](https://docs.djangoproject.com/en/6.0/ref/forms/api/#dynamic-initial-values) — Form customization documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*