---
source_course: "django-deployment"
source_lesson: "django-deployment-gunicorn-setup"
---

# Gunicorn WSGI Server

Gunicorn (Green Unicorn) is a Python WSGI HTTP Server for production.

## Basic Setup

```bash
# Install
pip install gunicorn

# Run
gunicorn myproject.wsgi:application

# With options
gunicorn myproject.wsgi:application \
    --bind 0.0.0.0:8000 \
    --workers 4
```

## Configuration File

```python
# gunicorn.conf.py
import multiprocessing

# Server socket
bind = '0.0.0.0:8000'
backlog = 2048

# Worker processes
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = 'sync'
worker_connections = 1000
timeout = 30
keepalive = 2

# Process naming
proc_name = 'myproject'

# Logging
accesslog = '-'  # stdout
errorlog = '-'   # stderr
loglevel = 'info'
access_log_format = '%(h)s %(l)s %(u)s %(t)s "%(r)s" %(s)s %(b)s "%(f)s" "%(a)s"'

# Security
limit_request_line = 4094
limit_request_fields = 100
limit_request_field_size = 8190
```

```bash
# Run with config file
gunicorn myproject.wsgi:application -c gunicorn.conf.py
```

## Worker Types

```python
# gunicorn.conf.py

# Sync workers (default) - Good for CPU-bound
worker_class = 'sync'
workers = (multiprocessing.cpu_count() * 2) + 1

# Async workers with gevent - Good for I/O-bound
# pip install gevent
worker_class = 'gevent'
worker_connections = 1000
workers = multiprocessing.cpu_count() * 2

# Async workers with eventlet
# pip install eventlet
worker_class = 'eventlet'

# Thread workers
worker_class = 'gthread'
threads = 4
```

## Systemd Service

```ini
# /etc/systemd/system/gunicorn.service
[Unit]
Description=Gunicorn daemon for myproject
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/myproject
ExecStart=/var/www/myproject/venv/bin/gunicorn \
    --access-logfile - \
    --workers 4 \
    --bind unix:/run/gunicorn/myproject.sock \
    myproject.wsgi:application
ExecReload=/bin/kill -s HUP $MAINPID
RuntimeDirectory=gunicorn
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable gunicorn
sudo systemctl start gunicorn
sudo systemctl status gunicorn
```

## Graceful Restarts

```bash
# Send HUP signal for graceful reload
kill -HUP $(cat /run/gunicorn/gunicorn.pid)

# Or via systemctl
sudo systemctl reload gunicorn
```

## Monitoring

```python
# gunicorn.conf.py

# StatsD integration
from gunicorn import instrument
statsd_host = 'localhost:8125'
statsd_prefix = 'myproject'

# Custom hooks
def on_starting(server):
    print('Starting Gunicorn server')

def on_exit(server):
    print('Shutting down Gunicorn server')

def worker_abort(worker):
    print(f'Worker {worker.pid} aborted')
```

## Resources

- [Gunicorn Documentation](https://docs.gunicorn.org/en/stable/) — Official Gunicorn docs

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*