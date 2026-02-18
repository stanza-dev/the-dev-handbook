---
source_course: "django-performance"
source_lesson: "django-performance-lazy-loading"
---

# Lazy Loading Images and Content

## Introduction

Lazy loading defers loading of non-critical resources until they're needed, improving initial page load time.

## Key Concepts

**Lazy Loading**: Load content when it enters viewport.

**Intersection Observer**: Browser API for visibility detection.

## Deep Dive

### Native Lazy Loading

```html
<!-- Modern browsers support native lazy loading -->
<img src="image.jpg" loading="lazy" alt="Description">
<iframe src="video.html" loading="lazy"></iframe>
```

### Django Template for Lazy Images

```python
# templatetags/image_tags.py
from django import template

register = template.Library()

@register.simple_tag
def lazy_image(image, alt='', css_class=''):
    return f'''
    <img src="{image.url}" 
         loading="lazy"
         alt="{alt}"
         class="{css_class}"
         width="{image.width}"
         height="{image.height}">
    '''
```

### Placeholder Images

```python
# Generate tiny placeholder
from PIL import Image
import base64
import io

def generate_placeholder(image_path, size=(10, 10)):
    img = Image.open(image_path)
    img.thumbnail(size)
    buffer = io.BytesIO()
    img.save(buffer, format='JPEG', quality=20)
    return base64.b64encode(buffer.getvalue()).decode()
```

```html
<img src="data:image/jpeg;base64,{{ placeholder }}"
     data-src="{{ image.url }}"
     class="lazy-load">
```

## Best Practices

1. **Use native loading="lazy"**: Best browser support.
2. **Set width and height**: Prevents layout shift.
3. **Provide placeholders**: Better user experience.

## Summary

Use native lazy loading for images and iframes. Always set dimensions to prevent layout shift. Consider placeholder images for better perceived performance.

## Resources

- [Lazy Loading](https://web.dev/lazy-loading/) — Web.dev lazy loading guide

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*