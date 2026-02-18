---
source_course: "django-deployment"
source_lesson: "django-deployment-uvicorn-asgi"
---

# Uvicorn ASGI Server

Uvicorn is a lightning-fast ASGI server for async Django.

## Basic Setup

```bash
# Install
pip install uvicorn

# Run
uvicorn myproject.asgi:application

# With options
uvicorn myproject.asgi:application \
    --host 0.0.0.0 \
    --port 8000 \
    --workers 4
```

## ASGI Configuration

```python
# myproject/asgi.py
import os
from django.core.asgi import get_asgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings.production')

application = get_asgi_application()
```

## Production Configuration

```bash
# Production command
uvicorn myproject.asgi:application \
    --host 0.0.0.0 \
    --port 8000 \
    --workers 4 \
    --loop uvloop \
    --http httptools \
    --log-level info \
    --access-log
```

## Gunicorn with Uvicorn Workers

```bash
# Best of both worlds
gunicorn myproject.asgi:application \
    -w 4 \
    -k uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000
```

```python
# gunicorn.conf.py
workers = 4
worker_class = 'uvicorn.workers.UvicornWorker'
bind = '0.0.0.0:8000'
```

## Daphne ASGI Server

```bash
# Alternative ASGI server (good for WebSockets)
pip install daphne

# Run
daphne -b 0.0.0.0 -p 8000 myproject.asgi:application

# With multiple workers
daphne -b 0.0.0.0 -p 8000 \
    --proxy-headers \
    myproject.asgi:application
```

## Systemd for Uvicorn

```ini
# /etc/systemd/system/uvicorn.service
[Unit]
Description=Uvicorn ASGI server for myproject
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/myproject
EnvironmentFile=/var/www/myproject/.env
ExecStart=/var/www/myproject/venv/bin/uvicorn \
    myproject.asgi:application \
    --host 0.0.0.0 \
    --port 8000 \
    --workers 4 \
    --loop uvloop
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

## When to Use ASGI vs WSGI

```python
# Use WSGI (Gunicorn) when:
# - Your app is mostly synchronous
# - You don't need WebSockets
# - Maximum compatibility is needed

# Use ASGI (Uvicorn/Daphne) when:
# - You have async views
# - You need WebSockets
# - You're using Django Channels
# - You want better concurrency for I/O-bound operations
```

## Resources

- [ASGI Servers](https://docs.djangoproject.com/en/6.0/howto/deployment/asgi/) — Django ASGI deployment

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*