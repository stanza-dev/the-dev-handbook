---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-form-lifecycle"
---

## Introduction

Every Django form follows a predictable lifecycle from instantiation through validation to data access. Understanding this lifecycle is the foundation for writing forms that behave exactly as you expect.

## Key Concepts

- **Bound form**: A form that has received user data via the `data` parameter.
- **Unbound form**: A form with no submitted data, for rendering only.
- **full_clean()**: Triggered by `is_valid()`, orchestrates all validation.
- **cleaned_data**: Dictionary of validated field values.
- **BoundField**: Pairs a field definition with its submitted value and errors.

## Real World Context

When debugging why a form silently rejects valid-looking input, the lifecycle diagram is your map. A registration form where `clean_username()` normalizes to lowercase while `clean()` verifies passwords match shows exactly where each check belongs.

## Deep Dive

# The Form Lifecycle

Understanding how Django processes forms helps you customize their behavior effectively.

## Form Initialization

```python
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)

# Unbound form (no data)
form = ContactForm()
print(form.is_bound)  # False

# Bound form (with data)
form = ContactForm(data={'name': 'John', 'email': 'john@example.com'})
print(form.is_bound)  # True

# Form with initial values (not bound!)
form = ContactForm(initial={'name': 'Default Name'})
print(form.is_bound)  # False
```

## The Validation Process

```
form.is_valid() called
        │
        ▼
┌───────────────────────┐
│   form.full_clean()   │
└───────────────────────┘
        │
        ├───────────────────────────┐
        ▼                           │
┌───────────────────────┐           │
│  _clean_fields()      │           │
│  For each field:      │           │
│   1. field.clean()    │ ──► field validation
│   2. clean_<field>()  │ ──► custom field validation
└───────────────────────┘           │
        │                           │
        ▼                           │
┌───────────────────────┐           │
│  _clean_form()        │           │
│   • form.clean()      │ ──► cross-field validation
└───────────────────────┘           │
        │                           │
        ▼                           │
┌───────────────────────┐           │
│ _post_clean()         │           │
│ (ModelForm specific)  │           │
└───────────────────────┘           │
        │                           │
        ▼                           │
   form.cleaned_data ◄──────────────┘
   form.errors
```

## Field Validation Order

```python
class RegistrationForm(forms.Form):
    username = forms.CharField(max_length=30)
    email = forms.EmailField()
    password = forms.CharField(widget=forms.PasswordInput)
    confirm_password = forms.CharField(widget=forms.PasswordInput)
    
    def clean_username(self):
        """Called after field.clean() for username."""
        username = self.cleaned_data['username']
        if User.objects.filter(username=username).exists():
            raise forms.ValidationError('Username already taken')
        return username
    
    def clean_email(self):
        """Called after field.clean() for email."""
        email = self.cleaned_data['email']
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError('Email already registered')
        return email
    
    def clean(self):
        """Called after all field validations."""
        cleaned_data = super().clean()
        password = cleaned_data.get('password')
        confirm = cleaned_data.get('confirm_password')
        
        if password and confirm and password != confirm:
            raise forms.ValidationError('Passwords do not match')
        
        return cleaned_data
```

## Accessing Form Data

```python
def contact_view(request):
    form = ContactForm(request.POST or None)
    
    if form.is_valid():
        # Access validated data
        name = form.cleaned_data['name']
        email = form.cleaned_data['email']
        
        # cleaned_data only exists after is_valid()
        print(form.cleaned_data)
    else:
        # Access errors
        print(form.errors)  # Dict-like: {'field': ['error1', 'error2']}
        print(form.errors.as_json())  # JSON format
        print(form.non_field_errors())  # Errors from clean()
```

## BoundField Objects

```python
form = ContactForm(data={'name': 'John'})

# Access bound fields
name_field = form['name']  # BoundField object

print(name_field.name)         # 'name'
print(name_field.value())      # 'John'
print(name_field.label)        # 'Name'
print(name_field.id_for_label) # 'id_name'
print(name_field.errors)       # []
print(name_field.is_hidden)    # False
print(name_field.html_name)    # 'name'

# Iterate over form fields
for field in form:
    print(f"{field.label}: {field.value()}")
```

## Common Pitfalls

1. **Confusing `initial` with `data`** -- `initial=` does NOT make a form bound. Only `data=` does.
2. **Accessing `cleaned_data` before `is_valid()`** -- Raises `AttributeError`.
3. **Forgetting to return from `clean_<field>()`** -- The field value becomes `None`.

## Best Practices

1. **Always call `super().clean()` first** -- Preserves parent validation including unique constraints.
2. **Use `clean_<field>()` for single-field, `clean()` for cross-field** -- Keeps errors associated with correct fields.
3. **Use `.get()` in `clean()`** -- Failed fields are absent from `cleaned_data`.

## Summary

- A form is **bound** when passed `data=`, **unbound** otherwise.
- `is_valid()` triggers `full_clean()` which runs `_clean_fields()`, `_clean_form()`, `_post_clean()`.
- `clean_<fieldname>()` runs per-field validation and must return the cleaned value.
- `clean()` handles cross-field validation and must return `cleaned_data`.
- `BoundField` objects provide field value, label, and errors in templates.

## Code Examples

**Creating bound and unbound forms -- a bound form has data to validate while an unbound form is just for display**

```python
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)

# Unbound form (no data yet)
form = ContactForm()
print(form.is_bound)  # False

# Bound form (with user data)
form = ContactForm(data={'name': 'Alice', 'email': 'alice@example.com'})
print(form.is_bound)  # True
print(form.is_valid())  # False (message is missing)
```


## Resources

- [Form API](https://docs.djangoproject.com/en/6.0/ref/forms/api/) — Complete Form API reference

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*