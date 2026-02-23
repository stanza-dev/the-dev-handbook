---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-widget-media"
---

# Widget CSS and JavaScript

## Introduction

Widgets can declare their CSS and JavaScript dependencies using the Media class.

## Key Concepts

- **Media class**: Declares CSS/JS deps on widgets.

## Real World Context

Widgets need their own CSS/JS. Media class handles deduplication.

## Deep Dive

### Defining Widget Media

```python
class DatePickerWidget(forms.DateInput):
    class Media:
        css = {'all': ('css/datepicker.css',)}
        js = ('js/datepicker.js',)
    
    def __init__(self, attrs=None):
        default_attrs = {'class': 'datepicker'}
        if attrs:
            default_attrs.update(attrs)
        super().__init__(attrs=default_attrs)
```

### Using Media in Templates

```html
<head>{{ form.media.css }}</head>
<body>
    {{ form.as_p }}
    {{ form.media.js }}
</body>
```

## Common Pitfalls

1. **Wrong placement** -- Separate css and js.
2. **Missing static config** -- Paths use STATIC_URL.

## Best Practices

1. **Media on widgets** -- Not forms.
2. **Use tuples** -- Convention.

## Summary

Use Media class for widget dependencies. Include form.media in templates.

## Resources

- [Widget Media](https://docs.djangoproject.com/en/6.0/topics/forms/media/) — Media assets

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*