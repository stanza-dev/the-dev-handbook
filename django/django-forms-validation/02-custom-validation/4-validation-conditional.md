---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-conditional"
---

# Conditional Validation

## Introduction

Sometimes validation rules depend on other field values or external conditions.

## Key Concepts

**Conditional Logic**: Rules that depend on other fields.

**Optional Requirements**: Required only under certain conditions.

## Deep Dive

### Field-Dependent Validation

```python
class ShippingForm(forms.Form):
    delivery_type = forms.ChoiceField(
        choices=[('pickup', 'Store Pickup'), ('delivery', 'Home Delivery')]
    )
    address = forms.CharField(required=False)
    
    def clean(self):
        cleaned_data = super().clean()
        delivery = cleaned_data.get('delivery_type')
        address = cleaned_data.get('address')
        
        # Address required only for delivery
        if delivery == 'delivery' and not address:
            self.add_error('address', 'Address required for delivery')
        
        return cleaned_data
```

### Dynamic Required Fields

```python
class PaymentForm(forms.Form):
    payment_method = forms.ChoiceField(
        choices=[('card', 'Credit Card'), ('bank', 'Bank Transfer')]
    )
    card_number = forms.CharField(required=False)
    bank_account = forms.CharField(required=False)
    
    def clean(self):
        cleaned_data = super().clean()
        method = cleaned_data.get('payment_method')
        
        if method == 'card' and not cleaned_data.get('card_number'):
            self.add_error('card_number', 'Required for card payments')
        elif method == 'bank' and not cleaned_data.get('bank_account'):
            self.add_error('bank_account', 'Required for bank transfers')
        
        return cleaned_data
```

## Summary

Use clean() for conditional validation. Check field dependencies. Add targeted errors with add_error().

## Resources

- [Form Validation](https://docs.djangoproject.com/en/6.0/ref/forms/validation/) — Validation guide

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*