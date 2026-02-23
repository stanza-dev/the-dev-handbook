---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-multiwidget"
---

# MultiWidget for Complex Inputs

## Introduction

MultiWidget combines multiple widgets into one, useful for complex data like phone numbers or date ranges.

## Key Concepts

- **MultiWidget**: Combines sub-widgets into one logical widget.
- **MultiValueField**: Validates and compresses sub-values.
- **decompress()**: Splits stored value into sub-widget list.
- **compress()**: Combines sub-values into one.

## Real World Context

A phone number field with separate area code, prefix, and line inputs. A date range with from/to pickers. MultiWidget keeps a single field in cleaned_data.

## Deep Dive

### Creating a MultiWidget

```python
class PhoneWidget(forms.MultiWidget):
    def __init__(self, attrs=None):
        widgets = [
            forms.TextInput(attrs={'size': 3, 'maxlength': 3}),
            forms.TextInput(attrs={'size': 3, 'maxlength': 3}),
            forms.TextInput(attrs={'size': 4, 'maxlength': 4}),
        ]
        super().__init__(widgets, attrs)
    
    def decompress(self, value):
        if value:
            return value.split('-')
        return ['', '', '']

class PhoneField(forms.MultiValueField):
    def __init__(self, **kwargs):
        fields = (
            forms.CharField(max_length=3),
            forms.CharField(max_length=3),
            forms.CharField(max_length=4),
        )
        super().__init__(fields=fields, widget=PhoneWidget, **kwargs)
    
    def compress(self, data_list):
        return '-'.join(data_list) if data_list else ''
```

## Common Pitfalls

1. **Mismatched widget/field count** -- Causes index errors.
2. **Returning None from decompress()** -- Must return list matching sub-widget count.

## Best Practices

1. **Descriptive sub-widget placeholders** -- Help users know what to enter.
2. **Handle partial input in compress()** -- Decide behavior for incomplete values.

## Summary

MultiWidget splits complex values into multiple inputs. Use decompress() to split and compress() to combine.

## Resources

- [MultiWidget](https://docs.djangoproject.com/en/6.0/ref/forms/widgets/#multiwidget) — MultiWidget documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*