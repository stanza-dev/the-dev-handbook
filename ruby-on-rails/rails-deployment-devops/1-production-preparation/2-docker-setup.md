---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-docker-setup"
---

# Docker Setup

Docker ensures consistent environments across development and production.

## Dockerfile

```dockerfile
# Dockerfile
FROM ruby:3.3-slim AS base

WORKDIR /rails

# Install dependencies
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
    curl \
    libpq-dev \
    libvips \
    && rm -rf /var/lib/apt/lists/*

# Build stage
FROM base AS build

RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
    build-essential \
    git \
    node-gyp \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Install gems
COPY Gemfile Gemfile.lock ./
RUN bundle config set --local deployment true && \
    bundle config set --local without 'development test' && \
    bundle install && \
    rm -rf ~/.bundle/ vendor/bundle/ruby/*/cache

# Copy application
COPY . .

# Precompile assets
RUN SECRET_KEY_BASE=dummy ./bin/rails assets:precompile

# Final stage
FROM base

COPY --from=build /rails /rails
COPY --from=build /usr/local/bundle /usr/local/bundle

# Run as non-root user
RUN useradd rails --create-home --shell /bin/bash && \
    chown -R rails:rails /rails
USER rails:rails

ENTRYPOINT ["/rails/bin/docker-entrypoint"]
EXPOSE 3000
CMD ["./bin/rails", "server"]
```

## Docker Entrypoint

```bash
#!/bin/bash
set -e

# Run migrations if needed
if [ "$RUN_MIGRATIONS" = "true" ]; then
  ./bin/rails db:migrate
fi

exec "$@"
```

## Docker Compose

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - '3000:3000'
    environment:
      - DATABASE_URL=postgres://postgres:password@db:5432/app_production
      - REDIS_URL=redis://redis:6379/1
      - RAILS_MASTER_KEY
    depends_on:
      - db
      - redis
  
  sidekiq:
    build: .
    command: bundle exec sidekiq
    environment:
      - DATABASE_URL=postgres://postgres:password@db:5432/app_production
      - REDIS_URL=redis://redis:6379/1
      - RAILS_MASTER_KEY
    depends_on:
      - db
      - redis
  
  db:
    image: postgres:16
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*