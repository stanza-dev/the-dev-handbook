---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-docker-setup"
---

# Docker for Rails

## Introduction
Docker ensures consistent environments across development and production. Rails 8 generates a production-ready Dockerfile automatically, using multi-stage builds and Thruster for HTTP compression and asset serving.

## Key Concepts
- **Multi-Stage Build**: A Dockerfile pattern that uses a build stage for compilation and a final stage with only runtime dependencies, producing smaller images.
- **Thruster**: A lightweight HTTP proxy included in Rails 8 that handles gzip compression, X-Sendfile, and cache headers — replacing the need for nginx in simple deployments.
- **Docker Entrypoint**: A script that runs before the main process, typically used to run database migrations on deploy.

## Real World Context
Rails 8 generates a production Dockerfile when you create a new app. This Dockerfile uses multi-stage builds to keep the final image small and includes Thruster for serving assets efficiently — no separate nginx or CDN needed for most apps.

## Deep Dive

### Rails 8 Dockerfile

```dockerfile
FROM ruby:3.4-slim AS base
WORKDIR /rails

RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
    curl libpq-dev libvips \
    && rm -rf /var/lib/apt/lists/*

FROM base AS build

RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
    build-essential git pkg-config \
    && rm -rf /var/lib/apt/lists/*

COPY Gemfile Gemfile.lock ./
RUN bundle config set --local deployment true && \
    bundle config set --local without 'development test' && \
    bundle install && \
    rm -rf ~/.bundle/ vendor/bundle/ruby/*/cache

COPY . .
RUN SECRET_KEY_BASE=dummy ./bin/rails assets:precompile

FROM base
COPY --from=build /rails /rails
COPY --from=build /usr/local/bundle /usr/local/bundle

RUN useradd rails --create-home --shell /bin/bash && \
    chown -R rails:rails /rails
USER rails:rails

ENTRYPOINT ["/rails/bin/docker-entrypoint"]
EXPOSE 3000
# Thruster wraps the Rails server for compression and asset serving
CMD ["./bin/thrust", "./bin/rails", "server"]
```

### Docker Compose (Rails 8 — No Redis Needed)

```yaml
services:
  web:
    build: .
    ports:
      - '3000:3000'
    environment:
      - DATABASE_URL=postgres://postgres:password@db:5432/app_production
      - RAILS_MASTER_KEY
      - SOLID_QUEUE_IN_PUMA=true
    depends_on:
      - db

  db:
    image: postgres:17
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password

volumes:
  postgres_data:
```

Notice: no Redis service, no separate Sidekiq worker. Solid Queue runs embedded in Puma.

### Docker Entrypoint

```bash
#!/bin/bash
set -e

if [ "$RUN_MIGRATIONS" = "true" ]; then
  ./bin/rails db:migrate
fi

exec "$@"
```

## Common Pitfalls
1. **Adding a separate Sidekiq container when using Solid Queue** — With `SOLID_QUEUE_IN_PUMA=true`, jobs run inside the web process. No separate worker container needed for most apps.
2. **Using an old Ruby base image** — Rails 8.1 requires Ruby 3.2+. Use `ruby:3.4-slim` for the latest stable release.

## Best Practices
1. **Use Thruster in the CMD** — `CMD ["./bin/thrust", "./bin/rails", "server"]` wraps your server with compression and efficient asset serving.
2. **Keep Docker Compose simple** — Rails 8's Solid Trifecta means you only need a database container for most apps.

## Summary
- Rails 8 generates a production-ready Dockerfile with multi-stage builds.
- Thruster handles HTTP compression and asset serving — no nginx needed.
- Docker Compose only needs a database — no Redis or separate worker containers.
- Use `SOLID_QUEUE_IN_PUMA=true` to run background jobs inside the web process.

## Code Examples

**Thruster wraps the Rails server to handle compression and asset serving in production**

```bash
# Rails 8 production Docker CMD uses Thruster
CMD ["./bin/thrust", "./bin/rails", "server"]

# Thruster provides:
# - Gzip/Brotli compression for all responses
# - X-Sendfile for efficient file serving
# - Cache headers for fingerprinted assets
# - No nginx or separate proxy needed
```


## Resources

- [Rails Docker Guide](https://guides.rubyonrails.org/getting_started.html#deploying-to-production) — Official Rails getting started guide covering Docker and production deployment

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*