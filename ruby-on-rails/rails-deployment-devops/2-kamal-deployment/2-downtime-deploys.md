---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-zero-downtime-deploys"
---

# Zero-Downtime Deployments

## Introduction
Kamal 2's built-in proxy handles zero-downtime deployments automatically. It starts the new container, waits for it to pass health checks, then seamlessly shifts traffic from the old container to the new one.

## Key Concepts
- **Health Check**: An HTTP endpoint Kamal Proxy hits to verify the new container is ready to serve traffic before routing requests to it.
- **Rolling Deploy**: Updating servers one at a time so some servers always serve traffic during deployment.
- **Rollback**: Reverting to the previous container version if the new deploy fails health checks.

## Real World Context
With zero-downtime deployment, your users never see a maintenance page or experience dropped connections during deploys. Kamal Proxy holds existing connections on the old container while routing new ones to the new container.

## Deep Dive

### Health Check Configuration

```yaml
# config/deploy.yml
proxy:
  ssl: true
  host: myapp.com
  healthcheck:
    path: /health
    interval: 3
    timeout: 3
```

```ruby
# app/controllers/health_controller.rb
class HealthController < ApplicationController
  skip_before_action :authenticate_user!

  def show
    ActiveRecord::Base.connection.execute('SELECT 1')
    head :ok
  rescue StandardError
    head :service_unavailable
  end
end
```

```ruby
# config/routes.rb
get '/health', to: 'health#show'
```

### Database Migrations Before Deploy

```bash
# Run migrations on the server before deploying new code
kamal app exec 'bin/rails db:migrate'
kamal deploy
```

Or use a deploy hook:

```yaml
# config/deploy.yml
servers:
  web:
    hosts:
      - 192.168.1.1
      - 192.168.1.2
```

### Rollback

```bash
# Kamal auto-rolls back if health check fails
# Manual rollback to previous version:
kamal rollback
```

### Container Retention

```yaml
# config/deploy.yml
retain_containers: 5  # Keep 5 old versions for rollback
```

## Common Pitfalls
1. **Health check endpoint that requires authentication** — The health check path must be publicly accessible. Always `skip_before_action :authenticate_user!`.
2. **Destructive migrations before deploy** — If you remove a column that the old code still uses, the old containers will crash during the rolling deploy.

## Best Practices
1. **Keep health checks lightweight** — A simple SELECT 1 is sufficient. Don't check every dependency in the health check — use a separate readiness endpoint for detailed checks.
2. **Make migrations backwards-compatible** — The old and new code versions run simultaneously during a rolling deploy. Never remove a column until the old code is fully replaced.

## Summary
- Kamal Proxy handles zero-downtime deploys by starting new containers before stopping old ones.
- Health checks verify the new container is ready before routing traffic to it.
- Kamal automatically rolls back if the new container fails health checks.
- Migrations must be backwards-compatible with the currently running code.

## Code Examples

**A lightweight health check that Kamal Proxy uses to verify the app can serve requests**

```ruby
# Minimal health check endpoint
class HealthController < ApplicationController
  skip_before_action :authenticate_user!

  def show
    ActiveRecord::Base.connection.execute('SELECT 1')
    head :ok
  rescue StandardError
    head :service_unavailable
  end
end

# config/routes.rb
get '/health', to: 'health#show'
```


## Resources

- [Kamal Proxy Configuration](https://kamal-deploy.org/docs/configuration/proxy/) — Official Kamal documentation on proxy and health check configuration

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*