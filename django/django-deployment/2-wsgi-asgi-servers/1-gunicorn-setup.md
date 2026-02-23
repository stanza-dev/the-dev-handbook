---
source_course: "django-deployment"
source_lesson: "django-deployment-gunicorn-setup"
---

# Gunicorn WSGI Server

## Introduction

Gunicorn (Green Unicorn) is the most popular Python WSGI HTTP server for production Django deployments. It manages a pool of worker processes that handle incoming requests, providing the concurrency and reliability that Django's built-in development server cannot.

## Key Concepts

- **WSGI**: Web Server Gateway Interface — the standard protocol between Python web applications and web servers.
- **Worker Process**: An independent OS process that handles incoming HTTP requests. Multiple workers enable concurrency.
- **Worker Types**: sync (default, one request per worker), gevent (async I/O), gthread (threaded) — each suited to different workloads.
- **Graceful Restart**: Reloading worker processes without dropping active connections (via HUP signal).

## Real World Context

Gunicorn is used by the majority of Django applications in production, from small startups to large-scale services. The (2 x CPU cores) + 1 worker formula is a well-tested starting point, but teams often adjust based on monitoring. CPU-bound applications (image processing, data analysis) benefit from sync workers, while API-heavy applications with many external calls perform better with gevent workers.

## Deep Dive

Gunicorn is the most widely used WSGI server for Django in production. Proper configuration of worker count, timeouts, and logging is essential for reliable deployments.


## Introduction

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

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Too many workers** — Each worker consumes memory (typically 100-300MB for Django). On a 2GB server with 17 workers, you'll run out of memory. Monitor RSS per worker.
2. **Using the development server in production** — `python manage.py runserver` is single-threaded, not secure, and not designed for production traffic.
3. **Not setting a timeout** — Without `--timeout`, a hung request blocks a worker forever. Set `timeout = 30` to recycle stuck workers.

## Best Practices

1. **Start with (2 x CPU) + 1 workers** — This formula balances CPU utilization with I/O wait handling.
2. **Use a process manager** — Run Gunicorn under systemd or supervisord for automatic restarts and log management.
3. **Bind to a Unix socket** — Unix sockets are faster and more secure than TCP for communication between Nginx and Gunicorn on the same machine.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Gunicorn configuration file with worker count and logging settings**

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
# ...
```


## Resources

- [Gunicorn Documentation](https://docs.gunicorn.org/en/stable/) — Official Gunicorn docs

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*