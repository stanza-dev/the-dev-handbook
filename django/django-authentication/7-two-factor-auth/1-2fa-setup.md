---
source_course: "django-authentication"
source_lesson: "django-authentication-2fa-setup"
---

# Two-Factor Authentication Setup

Add TOTP (Time-based One-Time Password) authentication for enhanced security using django-two-factor-auth.

## Installation

```bash
pip install django-two-factor-auth
pip install phonenumbers  # For SMS backup
pip install qrcode        # For QR codes
```

## Configuration

```python
# settings.py
INSTALLED_APPS = [
    # ...
    'django_otp',
    'django_otp.plugins.otp_static',
    'django_otp.plugins.otp_totp',
    'two_factor',
    'two_factor.plugins.phonenumber',  # SMS backup
]

MIDDLEWARE = [
    # ...
    'django_otp.middleware.OTPMiddleware',
]

# Login URL
LOGIN_URL = 'two_factor:login'
LOGIN_REDIRECT_URL = 'two_factor:profile'

# Two-factor settings
TWO_FACTOR_CALL_GATEWAY = None  # For phone calls
TWO_FACTOR_SMS_GATEWAY = None   # For SMS
TWO_FACTOR_TOTP_DIGITS = 6
```

## URL Configuration

```python
# urls.py
from django.urls import path, include
from two_factor.urls import urlpatterns as tf_urls

urlpatterns = [
    path('', include(tf_urls)),
    # ...
]
```

## Protecting Views

```python
from django.contrib.auth.decorators import login_required
from django_otp.decorators import otp_required


@login_required
@otp_required
def sensitive_view(request):
    """Requires both login AND 2FA verification."""
    return render(request, 'sensitive.html')


# For class-based views
from django.contrib.auth.mixins import LoginRequiredMixin
from django_otp.decorators import otp_required
from django.utils.decorators import method_decorator


@method_decorator(otp_required, name='dispatch')
class SensitiveView(LoginRequiredMixin, View):
    def get(self, request):
        return render(request, 'sensitive.html')
```

## Manual 2FA Implementation

For custom implementations:

```python
# models.py
import pyotp
from django.db import models


class TwoFactorProfile(models.Model):
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE)
    totp_secret = models.CharField(max_length=32, blank=True)
    is_2fa_enabled = models.BooleanField(default=False)
    backup_codes = models.JSONField(default=list)
    
    def generate_secret(self):
        self.totp_secret = pyotp.random_base32()
        self.save()
        return self.totp_secret
    
    def get_totp_uri(self):
        return pyotp.totp.TOTP(self.totp_secret).provisioning_uri(
            name=self.user.email,
            issuer_name='MyApp'
        )
    
    def verify_code(self, code):
        totp = pyotp.TOTP(self.totp_secret)
        return totp.verify(code)
    
    def generate_backup_codes(self, count=10):
        import secrets
        codes = [secrets.token_hex(4) for _ in range(count)]
        self.backup_codes = codes
        self.save()
        return codes
```

```python
# views.py
import qrcode
import io
import base64


def setup_2fa(request):
    """Setup page for 2FA."""
    profile = request.user.twofactorprofile
    
    if request.method == 'POST':
        code = request.POST.get('code')
        if profile.verify_code(code):
            profile.is_2fa_enabled = True
            profile.save()
            backup_codes = profile.generate_backup_codes()
            return render(request, '2fa/backup_codes.html', {
                'codes': backup_codes
            })
        else:
            messages.error(request, 'Invalid code')
    
    # Generate new secret
    profile.generate_secret()
    
    # Create QR code
    qr = qrcode.make(profile.get_totp_uri())
    buffer = io.BytesIO()
    qr.save(buffer, format='PNG')
    qr_code = base64.b64encode(buffer.getvalue()).decode()
    
    return render(request, '2fa/setup.html', {
        'qr_code': qr_code,
        'secret': profile.totp_secret,
    })


def verify_2fa(request):
    """Verify 2FA code during login."""
    if request.method == 'POST':
        code = request.POST.get('code')
        profile = request.user.twofactorprofile
        
        # Check TOTP code
        if profile.verify_code(code):
            request.session['2fa_verified'] = True
            return redirect('dashboard')
        
        # Check backup codes
        if code in profile.backup_codes:
            profile.backup_codes.remove(code)
            profile.save()
            request.session['2fa_verified'] = True
            return redirect('dashboard')
        
        messages.error(request, 'Invalid code')
    
    return render(request, '2fa/verify.html')
```

## Setup Template

```html
<!-- templates/2fa/setup.html -->
<h1>Set Up Two-Factor Authentication</h1>

<p>Scan this QR code with your authenticator app:</p>
<img src="data:image/png;base64,{{ qr_code }}" alt="QR Code">

<p>Or manually enter this secret: <code>{{ secret }}</code></p>

<form method="post">
    {% csrf_token %}
    <label>Enter the 6-digit code from your app:</label>
    <input type="text" name="code" maxlength="6" pattern="[0-9]{6}" required>
    <button type="submit">Verify & Enable 2FA</button>
</form>
```

## Resources

- [django-two-factor-auth](https://django-two-factor-auth.readthedocs.io/) — Two-factor authentication for Django

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*