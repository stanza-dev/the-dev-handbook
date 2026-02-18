---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-zero-downtime-deploys"
---

# Zero Downtime Deploys

Ensure your application stays available during deployments.

## Health Check Configuration

```yaml
# config/deploy.yml
healthcheck:
  path: /health
  port: 3000
  max_attempts: 10
  interval: 5s
```

```ruby
# app/controllers/health_controller.rb
class HealthController < ApplicationController
  def show
    head :ok if healthy?
    head :service_unavailable unless healthy?
  end
  
  private
  
  def healthy?
    ActiveRecord::Base.connection.execute('SELECT 1')
    true
  rescue
    false
  end
end
```

## Rolling Deployment Strategy

```yaml
# config/deploy.yml
servers:
  web:
    hosts:
      - 192.168.1.1
      - 192.168.1.2
      - 192.168.1.3
    options:
      deploy:
        parallelism: 1  # Deploy one at a time
        delay: 10s       # Wait between each
```

## Database Migrations

```yaml
# config/deploy.yml
servers:
  web:
    hosts:
      - 192.168.1.1

# Run migrations before deploy
boot:
  limit: 1  # Only run on one server
  wait: 5

hooks:
  pre-deploy:
    - run: ./bin/rails db:migrate
```

Or use a separate migration job:

```bash
# Deploy with migrations
kamal app exec --reuse 'bin/rails db:migrate'
kamal deploy
```

## Rollback Strategy

```bash
# Rollback to previous version
kamal rollback

# Rollback to specific version
kamal rollback --version abc123

# Keep more versions available
# config/deploy.yml
retain_containers: 5
```

## Load Balancer Configuration

```yaml
traefik:
  options:
    publish:
      - '80:80'
      - '443:443'
  args:
    # Enable health checks
    entryPoints.web.address: ':80'
    entryPoints.websecure.address: ':443'
    # Sticky sessions if needed
    # providers.docker.defaultRule: "Host(`{{ .Name }}.example.com`)"
```

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*