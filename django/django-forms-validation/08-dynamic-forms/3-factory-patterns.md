---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-form-factory-patterns"
---

# Form Factory Patterns

## Introduction

When your application needs forms whose fields are entirely determined at runtime (like survey builders or CMS field editors), you need to create form classes dynamically. Python's type() function and Django's formfield_callback let you build these on the fly.

## Key Concepts

- **type()**: Python's built-in function for creating classes dynamically.
- **formfield_callback**: A hook in ModelForm that lets you customize how model fields become form fields.
- **Form class factory**: A function that returns a form class (not an instance) tailored to specific requirements.

## Real World Context

Survey platforms like Google Forms or Typeform let users define their own questions. The form that collects responses must be generated dynamically based on the survey definition. Similarly, CMS systems need dynamic forms for user-defined content types.

## Deep Dive

### Creating Form Classes with type()

Python's type() can create classes at runtime. A form class is just a class with form fields as attributes:

```python
from django import forms


def make_form_class(field_definitions):
    """Create a Form class from a list of field definitions."""
    fields = {}
    for field_def in field_definitions:
        if field_def['type'] == 'text':
            fields[field_def['name']] = forms.CharField(
                label=field_def['label'],
                required=field_def.get('required', True),
                max_length=field_def.get('max_length', 255),
            )
        elif field_def['type'] == 'email':
            fields[field_def['name']] = forms.EmailField(
                label=field_def['label'],
                required=field_def.get('required', True),
            )
        elif field_def['type'] == 'choice':
            fields[field_def['name']] = forms.ChoiceField(
                label=field_def['label'],
                choices=field_def['choices'],
            )
        elif field_def['type'] == 'number':
            fields[field_def['name']] = forms.IntegerField(
                label=field_def['label'],
                required=field_def.get('required', True),
            )

    # type(name, bases, attrs) creates a new class
    return type('DynamicForm', (forms.Form,), fields)
```

The key insight is that type('DynamicForm', (forms.Form,), fields) creates a class that inherits from forms.Form and has the specified fields as class attributes.

### Using the Factory in a View

```python
def survey_view(request, survey_id):
    survey = Survey.objects.get(pk=survey_id)
    field_defs = [
        {
            'name': f'field_{q.id}',
            'type': q.field_type,
            'label': q.text,
            'required': q.required,
            'choices': [(c.value, c.label) for c in q.choices.all()]
            if q.field_type == 'choice' else None,
        }
        for q in survey.questions.order_by('order')
    ]

    FormClass = make_form_class(field_defs)

    if request.method == 'POST':
        form = FormClass(request.POST)
        if form.is_valid():
            for key, value in form.cleaned_data.items():
                # Save each answer
                question_id = int(key.split('_')[1])
                Answer.objects.create(
                    survey=survey,
                    question_id=question_id,
                    value=str(value),
                )
            return redirect('survey_thanks')
    else:
        form = FormClass()

    return render(request, 'survey.html', {'form': form})
```

### formfield_callback for ModelForm Customization

formfield_callback lets you intercept how each model field becomes a form field:

```python
def custom_formfield(db_field, **kwargs):
    """Add Bootstrap classes to all form fields."""
    form_field = db_field.formfield(**kwargs)
    if form_field:
        form_field.widget.attrs['class'] = 'form-control'
    return form_field


class StyledArticleForm(forms.ModelForm):
    formfield_callback = custom_formfield

    class Meta:
        model = Article
        fields = ['title', 'body', 'status']
```

Every field on the form gets the 'form-control' class without specifying it per-field. This is especially useful when you have models with many fields.

## Common Pitfalls

1. **Losing validation across requests** — Dynamic form classes are created fresh each request. If you add custom clean methods, make sure they are included in the type() call or use a base class.
2. **Caching issues** — Don't cache form classes that depend on database state (like dynamic choices). The cached class would have stale choices.

## Best Practices

1. **Use a base class for shared validation** — Create a BaseSurveyForm with common clean() logic, then use it as the parent in type('DynamicForm', (BaseSurveyForm,), fields).
2. **Validate field definitions** — Always validate the field_definitions input before building the form class. Malformed definitions should be caught early, not at form render time.

## Summary

- Use type('FormName', (forms.Form,), fields_dict) to create form classes dynamically.
- Form factory functions return classes, not instances, keeping them reusable.
- formfield_callback customizes how model fields become form fields site-wide.
- Dynamic forms are ideal for surveys, CMS editors, and configurable data entry.

## Resources

- [Form API](https://docs.djangoproject.com/en/6.0/ref/forms/api/) — Django Form API reference

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*