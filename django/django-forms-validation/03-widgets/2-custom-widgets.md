---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-custom-widgets"
---

# Custom Widgets

Create custom widgets for special input types or enhanced functionality.

## Simple Custom Widget

```python
from django import forms


class ColorPickerWidget(forms.TextInput):
    """Widget for HTML5 color picker."""
    input_type = 'color'
    
    def __init__(self, attrs=None):
        default_attrs = {'class': 'color-picker'}
        if attrs:
            default_attrs.update(attrs)
        super().__init__(attrs=default_attrs)


# Usage
class ThemeForm(forms.Form):
    primary_color = forms.CharField(
        widget=ColorPickerWidget,
        initial='#007bff'
    )
```

## Widget with Custom Template

```python
class StarRatingWidget(forms.Widget):
    """Star rating input widget."""
    template_name = 'widgets/star_rating.html'
    
    def __init__(self, max_stars=5, attrs=None):
        self.max_stars = max_stars
        super().__init__(attrs)
    
    def get_context(self, name, value, attrs):
        context = super().get_context(name, value, attrs)
        context['widget'].update({
            'max_stars': self.max_stars,
            'range': range(1, self.max_stars + 1),
        })
        return context
    
    def format_value(self, value):
        return int(value) if value else 0
```

```html
<!-- templates/widgets/star_rating.html -->
<div class="star-rating" data-name="{{ widget.name }}">
    {% for i in widget.range %}
    <label>
        <input type="radio" 
               name="{{ widget.name }}" 
               value="{{ i }}"
               {% if i == widget.value %}checked{% endif %}>
        <span class="star">★</span>
    </label>
    {% endfor %}
</div>
```

## Multi-Value Widget

```python
class AddressWidget(forms.MultiWidget):
    """Composite widget for address input."""
    
    def __init__(self, attrs=None):
        widgets = [
            forms.TextInput(attrs={'placeholder': 'Street'}),
            forms.TextInput(attrs={'placeholder': 'City'}),
            forms.TextInput(attrs={'placeholder': 'State'}),
            forms.TextInput(attrs={'placeholder': 'ZIP'}),
        ]
        super().__init__(widgets, attrs)
    
    def decompress(self, value):
        """Split a single value into list for each widget."""
        if value:
            return value.split('|')
        return ['', '', '', '']


class AddressField(forms.MultiValueField):
    """Field that uses AddressWidget."""
    
    def __init__(self, **kwargs):
        fields = [
            forms.CharField(max_length=100),
            forms.CharField(max_length=50),
            forms.CharField(max_length=50),
            forms.CharField(max_length=10),
        ]
        super().__init__(fields, widget=AddressWidget, **kwargs)
    
    def compress(self, data_list):
        """Combine values from all widgets."""
        if data_list:
            return '|'.join(data_list)
        return ''


# Usage
class OrderForm(forms.Form):
    shipping_address = AddressField(required=True)
```

## Widget with JavaScript

```python
class AutocompleteWidget(forms.TextInput):
    """Text input with autocomplete functionality."""
    template_name = 'widgets/autocomplete.html'
    
    def __init__(self, data_url, attrs=None):
        self.data_url = data_url
        super().__init__(attrs)
    
    def get_context(self, name, value, attrs):
        context = super().get_context(name, value, attrs)
        context['widget']['data_url'] = self.data_url
        return context
    
    class Media:
        css = {
            'all': ('css/autocomplete.css',)
        }
        js = ('js/autocomplete.js',)
```

```html
<!-- templates/widgets/autocomplete.html -->
<div class="autocomplete-wrapper">
    <input type="{{ widget.type }}" 
           name="{{ widget.name }}" 
           {% if widget.value != None %}value="{{ widget.value }}"{% endif %}
           {% include "django/forms/widgets/attrs.html" %}
           data-autocomplete-url="{{ widget.data_url }}">
    <div class="autocomplete-results"></div>
</div>
```

## Using Widget Media

```html
<!-- In template -->
<head>
    {{ form.media }}  <!-- Includes CSS and JS for all widgets -->
    <!-- Or separately -->
    {{ form.media.css }}
</head>
<body>
    {{ form }}
    {{ form.media.js }}
</body>
```

## Resources

- [Custom Widgets](https://docs.djangoproject.com/en/6.0/ref/forms/widgets/#customizing-widget-instances) — Guide to creating custom widgets

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*