---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-kamal-introduction"
---

# Introduction to Kamal 2

## Introduction
Kamal 2 is Rails 8's built-in deployment tool. It deploys containerized applications to any Linux server with zero downtime. Kamal 2 replaces the generic Traefik proxy from Kamal 1 with its own purpose-built Kamal Proxy.

## Key Concepts
- **Kamal Proxy**: Kamal 2's built-in reverse proxy that handles SSL, health checks, and zero-downtime container swaps. Replaces Traefik from Kamal 1.
- **Accessories**: Supporting services (databases, Redis) managed by Kamal alongside your application.
- **Destinations**: Named deployment targets (staging, production) allowing different configurations for each environment.

## Real World Context
With Kamal 2, you can go from a fresh Ubuntu server to a fully deployed Rails application with SSL, zero-downtime deploys, and database services in under 10 minutes with a single `kamal setup` command.

## Deep Dive

### Installation

```bash
gem install kamal
kamal init
```

This creates `config/deploy.yml` and `.kamal/secrets`.

### Kamal 2 Configuration

```yaml
# config/deploy.yml
service: myapp
image: ghcr.io/myuser/myapp

servers:
  web:
    hosts:
      - 192.168.1.1
      - 192.168.1.2
  job:
    hosts:
      - 192.168.1.3
    cmd: bundle exec rake solid_queue:start

proxy:
  ssl: true
  host: myapp.com
  app_port: 3000

registry:
  server: ghcr.io
  username: myuser
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  clear:
    RAILS_ENV: production
    SOLID_QUEUE_IN_PUMA: true
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL

accessories:
  db:
    image: postgres:17
    host: 192.168.1.4
    port: 5432
    env:
      secret:
        - POSTGRES_PASSWORD
    volumes:
      - /data/postgres:/var/lib/postgresql/data
```

### Secrets File

```bash
# .kamal/secrets
KAMAL_REGISTRY_PASSWORD=ghp_xxx
RAILS_MASTER_KEY=abc123
DATABASE_URL=postgres://user:pass@db:5432/app
POSTGRES_PASSWORD=secure_password
```

### Deployment Commands

```bash
kamal setup    # First-time setup (installs Docker, deploys everything)
kamal deploy   # Deploy new version with zero downtime
kamal rollback # Roll back to previous version
kamal app logs # View application logs
kamal app exec 'bin/rails console' # Open Rails console
```

## Common Pitfalls
1. **Using Traefik configuration from Kamal 1** — Kamal 2 uses `proxy:` instead of `traefik:`. The proxy section is simpler and handles SSL automatically.
2. **Forgetting to set secrets** — Missing secrets in `.kamal/secrets` will cause deployment failures. Always verify with `kamal env push` before deploying.

## Best Practices
1. **Use `proxy: ssl: true`** — Kamal Proxy handles Let's Encrypt certificates automatically. No manual SSL configuration needed.
2. **Run Solid Queue in Puma for simple apps** — Set `SOLID_QUEUE_IN_PUMA: true` to avoid needing a separate job server.

## Summary
- Kamal 2 uses Kamal Proxy instead of Traefik for routing and SSL.
- Configuration lives in `config/deploy.yml` with secrets in `.kamal/secrets`.
- `kamal setup` handles first-time deployment from a fresh server.
- `kamal deploy` performs zero-downtime rolling updates.
- Accessories manage supporting services like databases.

## Code Examples

**Minimal Kamal 2 configuration — uses proxy: section instead of Kamal 1's traefik: section**

```yaml
# Kamal 2 deploy.yml — note proxy: instead of traefik:
service: myapp
image: ghcr.io/myuser/myapp

servers:
  web:
    hosts:
      - 192.168.1.1

proxy:
  ssl: true
  host: myapp.com

env:
  clear:
    SOLID_QUEUE_IN_PUMA: true
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL
```


## Resources

- [Kamal Documentation](https://kamal-deploy.org/docs/installation/) — Official Kamal 2 documentation covering installation, configuration, and deployment

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*