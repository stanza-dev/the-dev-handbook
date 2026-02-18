---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-deployment-strategies"
---

# Deployment Strategies

Different strategies suit different requirements.

## Blue-Green Deployment

Two identical environments, swap traffic after verification:

```yaml
# config/deploy.yml
servers:
  web:
    hosts:
      - blue.example.com
      - green.example.com
```

```bash
# Deploy to blue
kamal deploy --destination blue

# Test blue environment
curl https://blue.example.com/health

# Switch traffic to blue
kamal proxy rebalance
```

## Canary Deployment

Gradually shift traffic to new version:

```yaml
# config/deploy.yml
servers:
  web:
    hosts:
      - web1.example.com  # Canary
      - web2.example.com
      - web3.example.com
```

```bash
# Deploy to canary first
kamal app boot --hosts web1.example.com

# Monitor metrics
# If healthy, continue
kamal deploy
```

## Feature Flags

Deploy code but control activation:

```ruby
# Gemfile
gem 'flipper'
gem 'flipper-active_record'

# Usage
if Flipper.enabled?(:new_checkout, current_user)
  # New checkout flow
else
  # Old checkout flow
end

# Enable for percentage
Flipper.enable_percentage_of_actors(:new_checkout, 10)

# Enable for specific users
Flipper.enable(:new_checkout, current_user)
```

## Rollback Procedures

```bash
# Automatic rollback on health check failure
# (Built into Kamal)

# Manual rollback
kamal rollback

# Rollback to specific version
kamal rollback --version abc123

# List available versions
kamal app images
```

## Deployment Checklist

```yaml
# .github/workflows/deploy.yml
- name: Pre-deploy checks
  run: |
    # Verify migrations are safe
    bundle exec rails db:migrate:status
    
    # Check for pending migrations
    bundle exec rails runner 'exit(1) if ActiveRecord::Migration.check_pending!'
    
    # Verify assets compile
    bundle exec rails assets:precompile
```

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*