---
source_course: "django-deployment"
source_lesson: "django-deployment-compose-production"
---

# Docker Compose for Production

## Introduction

Docker Compose orchestrates multiple containers for local development and simple production deployments.

## Production Compose File

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  web:
    image: myregistry/myproject:${TAG:-latest}
    restart: always
    env_file:
      - .env.production
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
    healthcheck:
      test: curl -f http://localhost:8000/health/
      interval: 30s
      timeout: 10s
      retries: 3

  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - static_volume:/var/www/static:ro
    depends_on:
      - web

  db:
    image: postgres:15
    restart: always
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

volumes:
  postgres_data:
  static_volume:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Deployment Commands

```bash
# Build and push
docker-compose -f docker-compose.prod.yml build
docker-compose -f docker-compose.prod.yml push

# Deploy
docker-compose -f docker-compose.prod.yml up -d

# Rolling update
docker-compose -f docker-compose.prod.yml up -d --no-deps web
```

## Best Practices

1. **Separate files**: dev vs production compose files.
2. **Use secrets**: Don't put passwords in compose files.
3. **Health checks**: Enable automatic recovery.
4. **Resource limits**: Prevent runaway containers.

## Summary

Use separate compose files for production with health checks, secrets, resource limits, and restart policies. Consider Kubernetes for larger deployments.

## Resources

- [Compose in Production](https://docs.docker.com/compose/production/) — Docker Compose production guide

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*