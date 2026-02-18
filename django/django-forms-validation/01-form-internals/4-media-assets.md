---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-media-assets"
---

# Form Media and Assets

## Introduction

Forms can declare CSS and JavaScript dependencies via the Media class.

## Key Concepts

**Media Class**: Declares form CSS/JS requirements.

**Media Merging**: Combining media from multiple forms.

## Deep Dive

### Defining Media

```python
class DatePickerWidget(forms.TextInput):
    class Media:
        css = {
            'all': ('css/datepicker.css',)
        }
        js = ('js/datepicker.js',)

class MyForm(forms.Form):
    date = forms.DateField(widget=DatePickerWidget)
    
    class Media:
        css = {'all': ('css/forms.css',)}
        js = ('js/validation.js',)
```

### Using Media in Templates

```html
<head>
    {{ form.media.css }}
</head>
<body>
    {{ form.as_p }}
    {{ form.media.js }}
</body>
```

### Combining Media

```python
# Multiple forms
combined = form1.media + form2.media
```

## Summary

Use Media class to declare CSS/JS dependencies. Include form.media in templates. Media automatically combines from nested widgets.

## Resources

- [Form Media](https://docs.djangoproject.com/en/6.0/topics/forms/media/) — Form assets

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*