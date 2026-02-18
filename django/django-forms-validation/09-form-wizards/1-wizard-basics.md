---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-wizard-basics"
---

# Multi-Step Forms with django-formtools

Form wizards break long forms into manageable steps, improving user experience for complex data entry.

## Installation

```bash
pip install django-formtools
```

```python
# settings.py
INSTALLED_APPS = [
    # ...
    'formtools',
]
```

## Basic Wizard

```python
# forms.py
from django import forms


class PersonalInfoForm(forms.Form):
    first_name = forms.CharField(max_length=100)
    last_name = forms.CharField(max_length=100)
    email = forms.EmailField()


class AddressForm(forms.Form):
    street = forms.CharField(max_length=200)
    city = forms.CharField(max_length=100)
    state = forms.CharField(max_length=50)
    zip_code = forms.CharField(max_length=10)


class PreferencesForm(forms.Form):
    newsletter = forms.BooleanField(required=False)
    notifications = forms.ChoiceField(
        choices=[('all', 'All'), ('important', 'Important Only'), ('none', 'None')]
    )
```

```python
# views.py
from formtools.wizard.views import SessionWizardView


WIZARD_FORMS = [
    ('personal', PersonalInfoForm),
    ('address', AddressForm),
    ('preferences', PreferencesForm),
]


class RegistrationWizard(SessionWizardView):
    template_name = 'registration_wizard.html'
    form_list = WIZARD_FORMS
    
    def done(self, form_list, form_dict, **kwargs):
        """Called when all steps are complete."""
        # Combine all form data
        data = {}
        for form in form_list:
            data.update(form.cleaned_data)
        
        # Create user and profile
        user = User.objects.create_user(
            username=data['email'],
            email=data['email'],
            first_name=data['first_name'],
            last_name=data['last_name'],
        )
        
        Profile.objects.create(
            user=user,
            street=data['street'],
            city=data['city'],
            newsletter=data['newsletter'],
        )
        
        return redirect('registration_complete')
```

```python
# urls.py
from .views import RegistrationWizard, WIZARD_FORMS

urlpatterns = [
    path('register/', RegistrationWizard.as_view(WIZARD_FORMS), name='register'),
]
```

## Wizard Template

```html
<!-- templates/registration_wizard.html -->
{% extends 'base.html' %}

{% block content %}
<h1>Registration - Step {{ wizard.steps.step1 }} of {{ wizard.steps.count }}</h1>

<!-- Progress indicator -->
<div class="progress">
    {% for step in wizard.steps.all %}
        <span class="{% if step == wizard.steps.current %}active{% endif %}">
            {{ step }}
        </span>
    {% endfor %}
</div>

<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ wizard.management_form }}
    
    <h2>{{ wizard.steps.current }}</h2>
    {{ wizard.form.as_p }}
    
    <div class="navigation">
        {% if wizard.steps.prev %}
            <button name="wizard_goto_step" value="{{ wizard.steps.prev }}">
                Previous
            </button>
        {% endif %}
        
        <button type="submit">{% if wizard.steps.next %}Next{% else %}Submit{% endif %}</button>
    </div>
</form>
{% endblock %}
```

## Conditional Steps

```python
class ConditionalWizard(SessionWizardView):
    form_list = [
        ('basic', BasicInfoForm),
        ('business', BusinessInfoForm),  # Only for business accounts
        ('payment', PaymentForm),
    ]
    
    condition_dict = {
        'business': lambda wizard: wizard.get_cleaned_data_for_step('basic').get('account_type') == 'business'
    }
    
    def get_form_initial(self, step):
        """Set initial data for a step."""
        initial = super().get_form_initial(step)
        
        if step == 'payment':
            basic_data = self.get_cleaned_data_for_step('basic')
            if basic_data:
                initial['email'] = basic_data.get('email')
        
        return initial
```

## Accessing Previous Step Data

```python
class MyWizard(SessionWizardView):
    def get_context_data(self, form, **kwargs):
        context = super().get_context_data(form, **kwargs)
        
        # Access all previous data
        all_data = {}
        for step in self.get_form_list():
            step_data = self.get_cleaned_data_for_step(step)
            if step_data:
                all_data.update(step_data)
        
        context['previous_data'] = all_data
        return context
```

## File Uploads in Wizard

```python
from formtools.wizard.views import SessionWizardView
from django.core.files.storage import default_storage


class FileWizard(SessionWizardView):
    # Use file storage for wizard files
    file_storage = default_storage
    
    def done(self, form_list, **kwargs):
        # Access uploaded files
        for form in form_list:
            if hasattr(form, 'files'):
                for field, file in form.files.items():
                    # Process file
                    pass
```

## Resources

- [Form Wizard](https://django-formtools.readthedocs.io/en/latest/wizard.html) — django-formtools wizard documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*