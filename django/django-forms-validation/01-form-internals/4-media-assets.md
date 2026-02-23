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

## Real World Context

When building a form with a custom date picker widget and a rich text editor, each widget needs its own CSS and JavaScript files. The Media class automates this: include `{{ form.media }}` in your template and Django handles deduplication and ordering.

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

## Common Pitfalls

1. **Including `form.media` inside the form tag** -- CSS links must go in `<head>` and scripts before `</body>`. Use `form.media.css` and `form.media.js` separately.
2. **Duplicate assets from multiple forms** -- Use `form1.media + form2.media` which automatically deduplicates.
3. **Forgetting to serve static files** -- Media paths are relative to STATIC_URL. If static files are not configured, the assets will 404.

## Best Practices

1. **Split CSS and JS placement** -- Use `{{ form.media.css }}` in `<head>` and `{{ form.media.js }}` before `</body>`.
2. **Use the Media class on custom widgets, not forms** -- This keeps assets scoped correctly when the widget is reused.

## Summary

Use Media class to declare CSS/JS dependencies. Include form.media in templates. Media automatically combines from nested widgets.

## Resources

- [Form Media](https://docs.djangoproject.com/en/6.0/topics/forms/media/) — Form assets

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*