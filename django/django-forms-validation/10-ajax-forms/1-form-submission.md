---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-ajax-form-submission"
---

# AJAX Form Submission

Submit forms and handle responses without page reloads for a smoother user experience.

## Basic AJAX View

```python
# views.py
from django.http import JsonResponse
from django.views.decorators.http import require_POST
from .forms import ContactForm


@require_POST
def contact_ajax(request):
    form = ContactForm(request.POST)
    
    if form.is_valid():
        # Process form
        form.save()
        return JsonResponse({
            'success': True,
            'message': 'Thank you for your message!'
        })
    else:
        return JsonResponse({
            'success': False,
            'errors': form.errors
        }, status=400)
```

## JavaScript Form Handler

```html
<!-- template.html -->
<form id="contact-form" method="post">
    {% csrf_token %}
    {{ form.as_div }}
    <button type="submit">Send</button>
</form>

<div id="form-messages"></div>

<script>
const form = document.getElementById('contact-form');
const messages = document.getElementById('form-messages');

form.addEventListener('submit', async function(e) {
    e.preventDefault();
    
    // Clear previous errors
    document.querySelectorAll('.error-message').forEach(el => el.remove());
    
    const formData = new FormData(form);
    
    try {
        const response = await fetch(form.action, {
            method: 'POST',
            body: formData,
            headers: {
                'X-Requested-With': 'XMLHttpRequest',
            },
        });
        
        const data = await response.json();
        
        if (data.success) {
            messages.innerHTML = `<div class="success">${data.message}</div>`;
            form.reset();
        } else {
            displayErrors(data.errors);
        }
    } catch (error) {
        messages.innerHTML = '<div class="error">An error occurred. Please try again.</div>';
    }
});

function displayErrors(errors) {
    for (const [field, fieldErrors] of Object.entries(errors)) {
        const input = form.querySelector(`[name="${field}"]`);
        if (input) {
            const errorDiv = document.createElement('div');
            errorDiv.className = 'error-message';
            errorDiv.textContent = fieldErrors.join(', ');
            input.parentNode.appendChild(errorDiv);
        }
    }
}
</script>
```

## Class-Based AJAX View

```python
from django.http import JsonResponse
from django.views.generic import View
from django.views.decorators.csrf import csrf_exempt
from django.utils.decorators import method_decorator
import json


class AjaxFormView(View):
    """Base view for AJAX form handling."""
    form_class = None
    
    def post(self, request):
        # Handle JSON or form data
        if request.content_type == 'application/json':
            data = json.loads(request.body)
        else:
            data = request.POST
        
        form = self.form_class(data)
        
        if form.is_valid():
            result = self.form_valid(form)
            return JsonResponse(result)
        else:
            return JsonResponse({
                'success': False,
                'errors': form.errors
            }, status=400)
    
    def form_valid(self, form):
        """Override to handle valid form."""
        form.save()
        return {'success': True}


class ContactAjaxView(AjaxFormView):
    form_class = ContactForm
    
    def form_valid(self, form):
        contact = form.save()
        send_notification_email(contact)
        return {
            'success': True,
            'message': 'Message sent successfully!',
            'id': contact.pk
        }
```

## Inline Validation

```python
# views.py
from django.http import JsonResponse


def validate_username(request):
    """Check username availability."""
    username = request.GET.get('username', '')
    
    if len(username) < 3:
        return JsonResponse({
            'valid': False,
            'message': 'Username must be at least 3 characters'
        })
    
    exists = User.objects.filter(username=username).exists()
    
    return JsonResponse({
        'valid': not exists,
        'message': 'Username taken' if exists else 'Username available'
    })
```

```javascript
// Inline validation on blur
const usernameInput = document.getElementById('id_username');

usernameInput.addEventListener('blur', async function() {
    const username = this.value;
    if (username.length < 3) return;
    
    const response = await fetch(`/validate-username/?username=${encodeURIComponent(username)}`);
    const data = await response.json();
    
    const feedback = this.nextElementSibling;
    feedback.textContent = data.message;
    feedback.className = data.valid ? 'valid' : 'invalid';
});
```

## CSRF Token Handling

```javascript
// Get CSRF token from cookie
function getCookie(name) {
    let cookieValue = null;
    if (document.cookie && document.cookie !== '') {
        const cookies = document.cookie.split(';');
        for (let i = 0; i < cookies.length; i++) {
            const cookie = cookies[i].trim();
            if (cookie.substring(0, name.length + 1) === (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}

const csrftoken = getCookie('csrftoken');

// Include in fetch requests
fetch('/api/submit/', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRFToken': csrftoken,
    },
    body: JSON.stringify(data),
});
```

## Loading States

```javascript
async function submitForm(form) {
    const submitBtn = form.querySelector('button[type="submit"]');
    const originalText = submitBtn.textContent;
    
    // Show loading state
    submitBtn.disabled = true;
    submitBtn.textContent = 'Submitting...';
    
    try {
        const response = await fetch(form.action, {
            method: 'POST',
            body: new FormData(form),
        });
        
        const data = await response.json();
        // Handle response...
    } finally {
        // Restore button
        submitBtn.disabled = false;
        submitBtn.textContent = originalText;
    }
}
```

## Resources

- [AJAX in Django](https://docs.djangoproject.com/en/6.0/howto/csrf/#ajax) — CSRF handling for AJAX

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*