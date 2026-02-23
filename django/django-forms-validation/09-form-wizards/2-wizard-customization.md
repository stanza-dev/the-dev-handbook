---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-wizard-customization"
---

# Wizard Customization

## Introduction

Form wizards support conditional steps, pre-populated fields, and custom context data. These features let you build adaptive multi-step workflows that respond to user input at each step.

## Key Concepts

- **condition_dict**: A dictionary that controls which wizard steps are shown based on previous answers.
- **get_form_initial()**: Provides initial data for each step, often from a previous step's answers.
- **get_context_data()**: Adds custom data to the template context for any step.

## Real World Context

A loan application wizard might skip the business info step for personal accounts. An e-commerce checkout might skip the shipping step for digital products. Conditional steps prevent users from filling out irrelevant forms.

## Deep Dive

### Conditional Steps

Use condition_dict to show or skip steps based on previous data:

```python
from formtools.wizard.views import SessionWizardView


def needs_business_info(wizard):
    """Show business step only for business accounts."""
    data = wizard.get_cleaned_data_for_step('account') or {}
    return data.get('account_type') == 'business'


def needs_shipping(wizard):
    """Show shipping step only for physical products."""
    data = wizard.get_cleaned_data_for_step('product') or {}
    return data.get('is_physical', True)


class CheckoutWizard(SessionWizardView):
    form_list = [
        ('account', AccountForm),
        ('business', BusinessInfoForm),
        ('product', ProductForm),
        ('shipping', ShippingForm),
        ('payment', PaymentForm),
    ]

    condition_dict = {
        'business': needs_business_info,
        'shipping': needs_shipping,
    }

    def done(self, form_list, form_dict, **kwargs):
        all_data = {}
        for form in form_list:
            all_data.update(form.cleaned_data)
        Order.objects.create(**all_data)
        return redirect('order_confirmation')
```

The condition functions receive the wizard instance and return True to show the step or False to skip it. Steps with no condition always appear.

### Pre-Populating Steps with get_form_initial()

Carry data forward from earlier steps:

```python
class RegistrationWizard(SessionWizardView):
    form_list = [
        ('credentials', CredentialsForm),
        ('profile', ProfileForm),
        ('preferences', PreferencesForm),
    ]

    def get_form_initial(self, step):
        initial = super().get_form_initial(step)

        if step == 'profile':
            cred_data = self.get_cleaned_data_for_step('credentials')
            if cred_data:
                # Pre-fill name from email
                email = cred_data.get('email', '')
                initial['display_name'] = email.split('@')[0]

        return initial
```

This creates a smoother experience by suggesting reasonable defaults based on what the user already entered.

### Adding Context with get_context_data()

Provide extra data to the wizard template:

```python
class OnboardingWizard(SessionWizardView):
    template_name = 'onboarding/wizard.html'
    form_list = [
        ('basics', BasicsForm),
        ('skills', SkillsForm),
        ('goals', GoalsForm),
    ]

    def get_context_data(self, form, **kwargs):
        context = super().get_context_data(form, **kwargs)

        # Add step descriptions
        step_info = {
            'basics': {
                'title': 'Tell us about yourself',
                'description': 'Basic profile information',
            },
            'skills': {
                'title': 'Your skills',
                'description': 'Select technologies you know',
            },
            'goals': {
                'title': 'Learning goals',
                'description': 'What do you want to achieve?',
            },
        }
        current = self.steps.current
        context['step_info'] = step_info.get(current, {})

        # Add summary of previous data
        context['all_previous'] = {}
        for step in self.steps.all:
            data = self.get_cleaned_data_for_step(step)
            if data:
                context['all_previous'][step] = data

        return context
```

This lets the template display a sidebar summary of previously entered data alongside the current step.

### Custom Step Templates

Use get_template_names() for per-step templates:

```python
class MultiTemplateWizard(SessionWizardView):
    def get_template_names(self):
        return [f'wizard/{self.steps.current}.html']
```

Each step renders with its own template (wizard/account.html, wizard/payment.html, etc.).

## Common Pitfalls

1. **Condition functions returning None** — If get_cleaned_data_for_step() returns None (step not yet visited), your condition must handle that gracefully. Always default to True or False explicitly.
2. **Template context not updating** — get_context_data() is called on every render but cached step data only exists after validation. Check for None before accessing cleaned data.

## Best Practices

1. **Use named steps** — Define form_list as a list of (name, FormClass) tuples rather than bare classes. Named steps make condition_dict and get_cleaned_data_for_step() more readable.
2. **Show progress indicators** — Use wizard.steps.step1, wizard.steps.count, and wizard.steps.all in your template to build a visual progress bar.

## Summary

- condition_dict controls which steps are shown based on previous answers.
- get_form_initial() carries data forward from earlier steps.
- get_context_data() provides extra template data like step descriptions and progress info.
- get_template_names() allows per-step custom templates.

## Resources

- [Form Wizard Customization](https://django-formtools.readthedocs.io/en/latest/wizard.html#advanced-wizardview-methods) — Advanced wizard configuration methods

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*