---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-cross-field-validation"
---

## Introduction

Cross-field validation uses `clean()` to enforce rules between multiple fields like date ranges or password confirmation.

## Key Concepts

- **clean()**: Runs after all field validations with full `cleaned_data`.
- **add_error()**: Associates errors with specific fields.
- **Non-field errors**: Form-wide errors from `form.non_field_errors()`.

## Real World Context

An event form ensures end date is after start date. A shipping form requires address only for delivery. These cross-field rules belong in `clean()`.

## Deep Dive

# Cross-Field Validation

The `clean()` method validates relationships between multiple fields.

## Basic Cross-Field Validation

```python
from django import forms
from django.core.exceptions import ValidationError


class DateRangeForm(forms.Form):
    start_date = forms.DateField()
    end_date = forms.DateField()
    
    def clean(self):
        cleaned_data = super().clean()
        start = cleaned_data.get('start_date')
        end = cleaned_data.get('end_date')
        
        # Only validate if both fields are valid
        if start and end:
            if end < start:
                raise ValidationError('End date must be after start date.')
            
            if (end - start).days > 365:
                raise ValidationError('Date range cannot exceed one year.')
        
        return cleaned_data
```

## Adding Errors to Specific Fields

```python
class PasswordChangeForm(forms.Form):
    current_password = forms.CharField(widget=forms.PasswordInput)
    new_password = forms.CharField(widget=forms.PasswordInput)
    confirm_password = forms.CharField(widget=forms.PasswordInput)
    
    def __init__(self, user, *args, **kwargs):
        self.user = user
        super().__init__(*args, **kwargs)
    
    def clean(self):
        cleaned_data = super().clean()
        current = cleaned_data.get('current_password')
        new = cleaned_data.get('new_password')
        confirm = cleaned_data.get('confirm_password')
        
        # Check current password
        if current and not self.user.check_password(current):
            self.add_error('current_password', 'Incorrect password.')
        
        # Check passwords match
        if new and confirm and new != confirm:
            self.add_error('confirm_password', 'Passwords do not match.')
        
        # Check new password is different
        if current and new and current == new:
            self.add_error('new_password', 'New password must be different.')
        
        return cleaned_data
```

## Multiple Validation Errors

```python
class EventForm(forms.Form):
    title = forms.CharField(max_length=200)
    start_datetime = forms.DateTimeField()
    end_datetime = forms.DateTimeField()
    max_attendees = forms.IntegerField(min_value=1)
    registration_deadline = forms.DateTimeField()
    
    def clean(self):
        cleaned_data = super().clean()
        errors = {}  # Collect all errors
        
        start = cleaned_data.get('start_datetime')
        end = cleaned_data.get('end_datetime')
        deadline = cleaned_data.get('registration_deadline')
        
        if start and end:
            if end <= start:
                errors['end_datetime'] = 'Must be after start time.'
            
            duration = (end - start).total_seconds() / 3600
            if duration > 24:
                errors['end_datetime'] = 'Event cannot exceed 24 hours.'
        
        if start and deadline:
            if deadline >= start:
                errors['registration_deadline'] = 'Must be before event start.'
        
        if errors:
            raise ValidationError(errors)
        
        return cleaned_data
```

## Conditional Validation

```python
class ShippingForm(forms.Form):
    SHIPPING_CHOICES = [
        ('pickup', 'Store Pickup'),
        ('delivery', 'Home Delivery'),
    ]
    
    shipping_method = forms.ChoiceField(choices=SHIPPING_CHOICES)
    
    # Delivery fields
    address = forms.CharField(required=False)
    city = forms.CharField(required=False)
    postal_code = forms.CharField(required=False)
    
    # Pickup fields
    store_location = forms.ChoiceField(
        choices=[('store1', 'Store 1'), ('store2', 'Store 2')],
        required=False
    )
    
    def clean(self):
        cleaned_data = super().clean()
        method = cleaned_data.get('shipping_method')
        
        if method == 'delivery':
            # Require delivery fields
            required_fields = ['address', 'city', 'postal_code']
            for field in required_fields:
                if not cleaned_data.get(field):
                    self.add_error(field, 'Required for delivery.')
        
        elif method == 'pickup':
            # Require pickup fields
            if not cleaned_data.get('store_location'):
                self.add_error('store_location', 'Select a store.')
        
        return cleaned_data
```

## Validation with External Data

```python
class DiscountCodeForm(forms.Form):
    code = forms.CharField(max_length=20)
    
    def clean_code(self):
        code = self.cleaned_data['code'].upper()
        
        try:
            discount = DiscountCode.objects.get(code=code)
        except DiscountCode.DoesNotExist:
            raise ValidationError('Invalid discount code.')
        
        if not discount.is_active:
            raise ValidationError('This code has expired.')
        
        if discount.uses >= discount.max_uses:
            raise ValidationError('This code has reached its usage limit.')
        
        # Store the discount object for use in the view
        self.discount = discount
        return code
```

## Common Pitfalls

1. **Not checking field existence** -- Use `.get()` since failed fields are absent.
2. **Single ValidationError for multiple issues** -- Use `add_error()` multiple times.
3. **Forgetting `super().clean()` in ModelForm** -- Breaks unique constraints.

## Best Practices

1. **Use `add_error(field, msg)`** -- Keeps errors next to correct field.
2. **Collect all errors** -- Show users everything at once.
3. **Keep `clean()` focused** -- Single-field rules go in `clean_<field>()`.

## Summary

- Override `clean()` for multi-field validation.
- Call `super().clean()` and use `.get()`.
- Use `add_error()` for field-specific errors.
- Collect all errors before returning.

## Resources

- [Cleaning and Validating Fields](https://docs.djangoproject.com/en/6.0/ref/forms/validation/#cleaning-and-validating-fields-that-depend-on-each-other) — Cross-field validation guide

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*