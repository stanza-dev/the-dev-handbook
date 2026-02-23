---
source_course: "django-foundations"
source_lesson: "django-foundations-wsgi-deployment"
---

# Deploying with WSGI

WSGI (Web Server Gateway Interface) is the standard for deploying Python web applications. Gunicorn is the most popular WSGI server for Django.

## Install Gunicorn

Gunicorn is the most popular WSGI server for Django. Install it into your virtual environment.

```bash
pip install gunicorn
```

## Basic Gunicorn Usage

Start Gunicorn by pointing it to your project's WSGI application. You can specify the bind address and number of worker processes.

```bash
# Start Gunicorn (from project directory)
gunicorn mysite.wsgi:application

# With options
gunicorn mysite.wsgi:application \
    --bind 0.0.0.0:8000 \
    --workers 3
```

## Calculating Workers

Recommended formula: `(2 x CPU cores) + 1`

```bash
# For a 2-core server
gunicorn mysite.wsgi:application --workers 5
```

## Gunicorn Configuration File

For production, define Gunicorn settings in a configuration file rather than passing them as command-line arguments.

```python
# gunicorn.conf.py
import multiprocessing

# Server socket
bind = '0.0.0.0:8000'

# Workers
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = 'sync'
worker_connections = 1000
timeout = 30
keepalive = 2

# Logging
accesslog = '/var/log/gunicorn/access.log'
errorlog = '/var/log/gunicorn/error.log'
loglevel = 'info'

# Process naming
proc_name = 'mysite'
```

```bash
gunicorn -c gunicorn.conf.py mysite.wsgi:application
```

## Using Nginx as Reverse Proxy

Nginx handles static files and forwards dynamic requests to Gunicorn:

```nginx
# /etc/nginx/sites-available/mysite
server {
    listen 80;
    server_name yourdomain.com;

    location /static/ {
        alias /path/to/staticfiles/;
    }

    location /media/ {
        alias /path/to/media/;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Systemd Service

Run Gunicorn as a system service:

```ini
# /etc/systemd/system/gunicorn.service
[Unit]
Description=Gunicorn daemon for mysite
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/path/to/mysite
ExecStart=/path/to/venv/bin/gunicorn \
    --workers 3 \
    --bind unix:/run/gunicorn/mysite.sock \
    mysite.wsgi:application

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
sudo systemctl enable gunicorn
sudo systemctl start gunicorn

# Check status
sudo systemctl status gunicorn

# View logs
sudo journalctl -u gunicorn
```

## Collecting Static Files

Before deploying, gather all static files from your apps into a single directory that your web server can serve.

```bash
# Collect all static files to STATIC_ROOT
python manage.py collectstatic
```

## Running Migrations in Production

Apply any pending database migrations as part of your deployment process. The `--no-input` flag prevents interactive prompts.

```bash
# Apply migrations
python manage.py migrate --no-input
```

## requirements.txt

Always maintain a requirements file:

```bash
# Generate
pip freeze > requirements.txt

# Install
pip install -r requirements.txt
```

```
# requirements.txt
Django>=6.0,<6.1
gunicorn>=21.0
psycopg2-binary>=2.9
whitenoise>=6.6
python-dotenv>=1.0
```

## Common Pitfalls

- **Using Django's development server in production**: `runserver` is single-threaded and not secure. Always use Gunicorn or uWSGI in production.
- **Forgetting to collect static files**: Without running `collectstatic`, your CSS, JS, and images will not be served in production.
- **Not setting up process management**: Gunicorn should run as a systemd service or via a process manager so it auto-restarts on crashes.

## Best Practices

- **Calculate Gunicorn workers** as `(2 * CPU cores) + 1` for optimal performance.
- **Use Nginx as a reverse proxy** in front of Gunicorn to handle SSL termination, static files, and load balancing.
- **Create a `requirements.txt`** with pinned versions to ensure reproducible deployments.

## Summary

- Use **Gunicorn** (or uWSGI) as the WSGI server instead of Django's development server
- Calculate workers as `(2 * CPU cores) + 1` for production
- Set up **Nginx as a reverse proxy** for SSL, static files, and load balancing
- Use **systemd** to run Gunicorn as a managed service with auto-restart
- Always run `collectstatic` and `migrate` as part of your deployment process

## Code Examples

**Production deployment commands: installing Gunicorn, collecting static files, and applying migrations**

```bash
# Install and run Gunicorn
pip install gunicorn
gunicorn mysite.wsgi:application --bind 0.0.0.0:8000 --workers 3

# Collect static files for production
python manage.py collectstatic --no-input

# Apply migrations
python manage.py migrate --no-input
```


## Resources

- [How to Deploy with WSGI](https://docs.djangoproject.com/en/6.0/howto/deployment/wsgi/) — Official WSGI deployment guide

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*