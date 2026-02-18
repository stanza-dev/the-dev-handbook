---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-multiwidget"
---

# MultiWidget for Complex Inputs

## Introduction

MultiWidget combines multiple widgets into one, useful for complex data like phone numbers or date ranges.

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

## Summary

MultiWidget splits complex values into multiple inputs. Use decompress() to split and compress() to combine.

## Resources

- [MultiWidget](https://docs.djangoproject.com/en/6.0/ref/forms/widgets/#multiwidget) — MultiWidget documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*