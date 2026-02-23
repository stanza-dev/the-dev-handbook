---
source_course: "django-deployment"
source_lesson: "django-deployment-docker-setup"
---

# Docker for Django

## Introduction

Docker ensures consistent environments across development, staging, and production by packaging your Django application with all its dependencies into a portable container. Multi-stage builds and proper security practices are essential for production-ready images.

## Key Concepts

- **Dockerfile**: A text file containing instructions to build a Docker image layer by layer.
- **Multi-Stage Build**: A Dockerfile technique that uses separate build and runtime stages, keeping the final image small and secure.
- **Docker Compose**: A tool for defining and running multi-container applications (Django + PostgreSQL + Redis + Nginx).
- **.dockerignore**: A file that excludes unnecessary files from the Docker build context, speeding up builds.

## Real World Context

Docker has become the standard deployment method for Django applications. It eliminates "works on my machine" problems by ensuring every environment runs identical software versions. Companies use Docker Compose for local development and Kubernetes or ECS for production orchestration. Multi-stage builds typically reduce image sizes from 1GB+ to 200-300MB.

## Deep Dive

Containerizing Django with Docker ensures consistent environments across development, staging, and production. A well-structured Dockerfile uses multi-stage builds, non-root users, and minimal base images.


## Introduction

Docker ensures consistent environments across development and production.

## Dockerfile

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PIP_NO_CACHE_DIR=1

# Set work directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libpq-dev \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

# Copy project
COPY . .

# Collect static files
RUN python manage.py collectstatic --noinput

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser
RUN chown -R appuser:appuser /app
USER appuser

# Run gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myproject.wsgi:application"]
```

## Multi-Stage Build

```dockerfile
# Dockerfile.multistage

# Build stage
FROM python:3.11-slim as builder

WORKDIR /app

RUN apt-get update && apt-get install -y \
    libpq-dev gcc

COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /app/wheels -r requirements.txt

# Final stage
FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && apt-get install -y libpq5 && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/wheels /wheels
RUN pip install --no-cache /wheels/*

COPY . .

RUN adduser --disabled-password appuser && \
    chown -R appuser:appuser /app
USER appuser

RUN python manage.py collectstatic --noinput

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myproject.wsgi:application"]
```

## Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DJANGO_SETTINGS_MODULE=myproject.settings.production
      - DATABASE_URL=postgres://postgres:postgres@db:5432/myproject
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media

  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=myproject
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres

  redis:
    image: redis:7-alpine

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - static_volume:/var/www/static
      - media_volume:/var/www/media
    depends_on:
      - web

  celery:
    build: .
    command: celery -A myproject worker -l info
    environment:
      - DJANGO_SETTINGS_MODULE=myproject.settings.production
      - DATABASE_URL=postgres://postgres:postgres@db:5432/myproject
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

volumes:
  postgres_data:
  static_volume:
  media_volume:
```

## Production Docker Compose

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  web:
    image: myregistry.com/myproject:${TAG:-latest}
    restart: always
    env_file:
      - .env.production
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health/"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## .dockerignore

```dockerfile
# .dockerignore
.git
.gitignore
.env*
*.pyc
__pycache__
.pytest_cache
.coverage
htmlcov
*.log
.vscode
.idea
node_modules
media/*
staticfiles/*
```

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Running as root** — Docker containers run as root by default. If compromised, an attacker has root access. Always create and switch to a non-root user.
2. **Not using .dockerignore** — Without it, the build context includes `.git`, `__pycache__`, `.env` files, and other unnecessary data, slowing builds and potentially leaking secrets.
3. **Using `latest` tag in production** — The `latest` tag is mutable and can point to different images over time. Always use specific version tags or git SHA for reproducible deployments.

## Best Practices

1. **Use multi-stage builds** — Separate build dependencies (gcc, dev headers) from runtime, reducing image size and attack surface.
2. **Pin base image versions** — Use `python:3.11.6-slim` not `python:3.11` to ensure reproducible builds.
3. **Run `collectstatic` in the build** — Include static file collection in the Dockerfile so the image is self-contained.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Multi-stage Dockerfile for Django with non-root user and static file collection**

```bash
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DJANGO_SETTINGS_MODULE=myproject.settings.production
      - DATABASE_URL=postgres://postgres:postgres@db:5432/myproject
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media

  db:
# ...
```


## Resources

- [Docker Documentation](https://docs.docker.com/) — Official Docker documentation

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*