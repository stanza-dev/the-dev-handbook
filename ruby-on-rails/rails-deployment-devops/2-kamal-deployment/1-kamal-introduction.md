---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-kamal-introduction"
---

# Kamal Introduction

Kamal deploys containerized applications to any server with zero-downtime.

## Installation

```bash
gem install kamal
```

## Initialization

```bash
kamal init
```

This creates `config/deploy.yml` and `.env` files.

## Configuration

```yaml
# config/deploy.yml
service: myapp

image: myregistry/myapp

servers:
  web:
    hosts:
      - 192.168.1.1
      - 192.168.1.2
    labels:
      traefik.http.routers.myapp.rule: Host(`myapp.com`)
  job:
    hosts:
      - 192.168.1.3
    cmd: bundle exec sidekiq

registry:
  server: ghcr.io
  username: myuser
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  clear:
    RAILS_ENV: production
    RAILS_LOG_TO_STDOUT: 'true'
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL
    - REDIS_URL

traefik:
  options:
    publish:
      - '443:443'
    volume:
      - '/letsencrypt:/letsencrypt'
  args:
    certificatesResolvers.letsencrypt.acme.email: admin@myapp.com
    certificatesResolvers.letsencrypt.acme.storage: /letsencrypt/acme.json
    certificatesResolvers.letsencrypt.acme.httpChallenge.entryPoint: web

accessories:
  db:
    image: postgres:16
    host: 192.168.1.4
    port: 5432
    env:
      secret:
        - POSTGRES_PASSWORD
    volumes:
      - /data/postgres:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    host: 192.168.1.4
    port: 6379
    volumes:
      - /data/redis:/data
```

## Environment Secrets

```bash
# .env
KAMAL_REGISTRY_PASSWORD=ghp_xxx
RAILS_MASTER_KEY=abc123
DATABASE_URL=postgres://user:pass@db:5432/app
REDIS_URL=redis://redis:6379/1
POSTGRES_PASSWORD=secure_password
```

## Deployment Commands

```bash
# Initial setup
kamal setup

# Deploy new version
kamal deploy

# Rollback
kamal rollback

# View logs
kamal app logs
```

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*