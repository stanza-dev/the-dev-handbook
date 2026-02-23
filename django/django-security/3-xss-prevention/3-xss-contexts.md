---
source_course: "django-security"
source_lesson: "django-security-xss-contexts"
---

# XSS in Different Contexts

## Introduction

XSS isn't just about HTML. User data can appear in JavaScript, URLs, CSS, and attributes—each requiring different escaping.

## Key Concepts

**HTML Context**: Most common—escaped by Django templates by default.

**JavaScript Context**: Inside <script> tags or event handlers.

**URL Context**: In href or src attributes.

**Attribute Context**: Inside HTML attribute values.



## Real World Context

Many XSS vulnerabilities occur not in HTML body text but in JavaScript contexts, URL attributes, or CSS. The Samy worm on MySpace exploited CSS expression injection, and countless sites have been compromised through href="javascript:..." in user-supplied URLs.

## Deep Dive

### JavaScript Context

```html
<!-- DANGEROUS -->
<script>var name = '{{ username }}';</script>

<!-- SAFE - use escapejs filter -->
<script>var name = '{{ username|escapejs }}';</script>

<!-- SAFEST - use JSON -->
<script>var data = {{ json_data|safe }};</script>
```

```python
# views.py
import json
from django.core.serializers.json import DjangoJSONEncoder

def view(request):
    return render(request, 'template.html', {
        'json_data': json.dumps(data, cls=DjangoJSONEncoder)
    })
```

### URL Context

```html
<!-- DANGEROUS -->
<a href="{{ user_url }}">Click</a>

<!-- Could be: javascript:alert('XSS') -->
```

```python
# Validate URLs
from django.core.validators import URLValidator

def validate_url(url):
    if url.startswith(('http://', 'https://')):
        URLValidator()(url)
        return url
    raise ValueError('Invalid URL scheme')
```

### Attribute Context

```html
<!-- DANGEROUS -->
<div class="{{ user_class }}">...</div>

<!-- Could inject: " onclick="alert('XSS') -->
```

```python
# Whitelist allowed values
def safe_class(value):
    allowed = ['primary', 'secondary', 'danger']
    return value if value in allowed else 'default'
```



## Common Pitfalls

1. **Using HTML escaping inside <script> tags** — HTML entities like &lt; are not JavaScript-safe; always use |escapejs or json.dumps() for JavaScript contexts.
2. **Allowing user-supplied URLs without scheme validation** — A href value of javascript:alert(1) bypasses HTML escaping entirely; always validate URL schemes.

## Best Practices

1. **Use context-appropriate escaping**: escapejs for JS, urlencode for URLs.
2. **Prefer JSON for data**: Properly handles all escaping.
3. **Whitelist, don't blacklist**: Only allow known-safe values.

## Summary

Different contexts need different escaping. Use escapejs for JavaScript, validate/whitelist URLs, and use json.dumps() for passing data to JavaScript safely.

## Resources

- [XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) — OWASP XSS Prevention Cheat Sheet

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*