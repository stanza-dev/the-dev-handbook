---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-github-actions"
---

# GitHub Actions for Rails

## Introduction
GitHub Actions automates testing and deployment. A well-configured CI pipeline catches bugs, security issues, and style violations before code reaches production.

## Key Concepts
- **Workflow**: A YAML file defining automated steps triggered by events like push or pull request.
- **Service Container**: A Docker container (PostgreSQL, Redis) that runs alongside your tests.
- **Matrix Strategy**: Running tests across multiple Ruby versions or configurations in parallel.

## Real World Context
Without CI, bugs slip into production because developers forget to run the full test suite locally. With GitHub Actions, every push automatically runs tests, linting, and security scans — nothing reaches main without passing.

## Deep Dive

### CI Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_PASSWORD: postgres
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    env:
      RAILS_ENV: test
      DATABASE_URL: postgres://postgres:postgres@localhost:5432/test

    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.4'
          bundler-cache: true
      - run: bin/rails db:setup
      - run: bin/rails test
      - run: bin/rails test:system

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.4'
          bundler-cache: true
      - run: bundle exec rubocop

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.4'
          bundler-cache: true
      - run: bundle exec bundler-audit check --update
      - run: bundle exec brakeman -q
```

### Deployment Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    needs: [test, lint, security]
    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.4'
      - run: gem install kamal
      - uses: docker/setup-buildx-action@v3
      - run: kamal deploy
        env:
          KAMAL_REGISTRY_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
          RAILS_MASTER_KEY: ${{ secrets.RAILS_MASTER_KEY }}
```

## Common Pitfalls
1. **Not using `bundler-cache: true`** — Without gem caching, every CI run installs all gems from scratch, adding 2-5 minutes.
2. **Running deploy without requiring CI to pass** — Always use `needs: [test, lint, security]` so deploys only happen after all checks pass.

## Best Practices
1. **Run lint, tests, and security in parallel** — These jobs are independent and can run simultaneously, cutting CI time.
2. **Cache Ruby gems with `bundler-cache: true`** — The ruby/setup-ruby action handles this automatically.

## Summary
- GitHub Actions automates tests, linting, and security scans on every push.
- Service containers provide PostgreSQL for tests without external dependencies.
- Deployment workflow uses `needs:` to require all checks before deploying.
- Parallel jobs and gem caching keep CI fast.

## Code Examples

**CI pipeline structure — tests, linting, and security run in parallel, deployment requires all to pass**

```yaml
# Parallel CI jobs — lint, test, and security run simultaneously
jobs:
  test:
    runs-on: ubuntu-latest
    # ...
  lint:
    runs-on: ubuntu-latest
    # ...
  security:
    runs-on: ubuntu-latest
    # ...
  deploy:
    needs: [test, lint, security]  # Only deploy if all pass
    # ...
```


## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions) — Official GitHub Actions documentation for CI/CD workflows

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*