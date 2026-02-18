---
source_course: "django-authentication"
source_lesson: "django-authentication-session-configuration"
---

# Session Configuration

Sessions store authentication state between requests. Configure them properly for security and performance.

## Session Backends

```python
# settings.py

# Database (default)
SESSION_ENGINE = 'django.contrib.sessions.backends.db'

# Cache
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

# Cached database (write-through)
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'

# File-based
SESSION_ENGINE = 'django.contrib.sessions.backends.file'

# Signed cookie (no server storage)
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'
```

## Session Settings

```python
# settings.py

# Session expiry
SESSION_COOKIE_AGE = 1209600  # 2 weeks in seconds (default)
SESSION_EXPIRE_AT_BROWSER_CLOSE = False  # True = session cookie

# Cookie settings
SESSION_COOKIE_NAME = 'sessionid'  # Cookie name
SESSION_COOKIE_DOMAIN = None       # None = current domain only
SESSION_COOKIE_PATH = '/'          # Cookie path

# Security settings
SESSION_COOKIE_SECURE = True       # Only send over HTTPS
SESSION_COOKIE_HTTPONLY = True     # No JavaScript access (default)
SESSION_COOKIE_SAMESITE = 'Lax'    # CSRF protection

# Save behavior
SESSION_SAVE_EVERY_REQUEST = False  # Only save if modified
```

## Using Sessions in Views

```python
def my_view(request):
    # Set session data
    request.session['favorite_color'] = 'blue'
    request.session['visit_count'] = request.session.get('visit_count', 0) + 1
    
    # Get session data
    color = request.session.get('favorite_color', 'red')
    
    # Delete session data
    del request.session['favorite_color']
    
    # Check if key exists
    if 'visit_count' in request.session:
        pass
    
    # Clear all session data
    request.session.flush()  # Also rotates session key
    
    # Set expiry
    request.session.set_expiry(300)  # 5 minutes
    request.session.set_expiry(0)    # Browser close
    request.session.set_expiry(None) # Use SESSION_COOKIE_AGE
```

## Cache-Based Sessions

For better performance:

```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}

SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
```

## Session Security Best Practices

```python
# settings.py

# For production
SESSION_COOKIE_SECURE = True      # HTTPS only
SESSION_COOKIE_HTTPONLY = True    # No JS access
SESSION_COOKIE_SAMESITE = 'Lax'   # CSRF protection

# Rotate session on login
from django.contrib.auth import login as auth_login

def login_view(request):
    # ... authenticate user ...
    auth_login(request, user)
    # Django automatically rotates session key on login
```

## Clearing Expired Sessions

```bash
# Run periodically (cron job)
python manage.py clearsessions
```

## Custom Session Engine Example

```python
# For Redis with custom serialization
from django.contrib.sessions.backends.base import SessionBase
import redis
import json

class RedisSession(SessionBase):
    def __init__(self, session_key=None):
        super().__init__(session_key)
        self._redis = redis.Redis(host='localhost', port=6379, db=0)
    
    def load(self):
        data = self._redis.get(self._get_redis_key())
        if data:
            return json.loads(data)
        return {}
    
    def save(self, must_create=False):
        key = self._get_redis_key()
        data = json.dumps(self._get_session(no_load=must_create))
        self._redis.setex(key, self.get_expiry_age(), data)
    
    def delete(self, session_key=None):
        self._redis.delete(self._get_redis_key(session_key))
    
    def _get_redis_key(self, session_key=None):
        return f'session:{session_key or self.session_key}'
```

## Resources

- [Sessions](https://docs.djangoproject.com/en/6.0/topics/http/sessions/) — Official sessions documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*