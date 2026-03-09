---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-deployment-strategies"
---

# Deployment Strategies

## Introduction
Different deployment strategies balance speed, safety, and complexity. From simple rolling deploys to canary releases, choosing the right strategy depends on your risk tolerance and infrastructure.

## Key Concepts
- **Rolling Deploy**: Update servers one at a time — Kamal's default strategy. Some servers run old code while others run new code during the transition.
- **Blue-Green Deploy**: Two identical environments — deploy to the inactive one, then swap traffic. Instant rollback by swapping back.
- **Canary Deploy**: Route a small percentage of traffic to the new version first, monitor for errors, then roll out fully.

## Real World Context
A SaaS company with 10,000 active users uses canary deployments to route 5% of traffic to new code first. If error rates spike, they roll back instantly without affecting 95% of users.

## Deep Dive

### Rolling Deploy (Kamal Default)

Kamal deploys to one server at a time:

```yaml
# config/deploy.yml
servers:
  web:
    hosts:
      - web1.example.com
      - web2.example.com
      - web3.example.com
```

```bash
kamal deploy  # Deploys to each host sequentially
```

### Blue-Green with Kamal Destinations

```yaml
# config/deploy.production.yml
servers:
  web:
    hosts:
      - blue.example.com
      - green.example.com
```

```bash
# Deploy and verify
kamal deploy --destination production
curl https://myapp.com/health
```

### Feature Flags

```ruby
# Gemfile
gem 'flipper'
gem 'flipper-active_record'

# Usage
if Flipper.enabled?(:new_checkout, current_user)
  render 'checkout/new'
else
  render 'checkout/legacy'
end

# Enable for percentage of users
Flipper.enable_percentage_of_actors(:new_checkout, 10)
```

### Rollback

```bash
# Kamal auto-rolls back on health check failure
# Manual rollback:
kamal rollback
```

## Common Pitfalls
1. **Deploying without feature flags for risky changes** — A feature flag lets you disable a broken feature without rolling back the entire deploy.
2. **Not monitoring after deploy** — Watch error rates and response times for 15-30 minutes after every deploy.

## Best Practices
1. **Use rolling deploys for most changes** — Kamal's default is safe and simple for the majority of deploys.
2. **Use feature flags for high-risk features** — Wrap new features in flags so you can disable them instantly without redeploying.

## Summary
- Rolling deploys (Kamal's default) update servers sequentially.
- Blue-green deployments allow instant rollback by swapping environments.
- Feature flags decouple deployment from feature activation.
- Always monitor error rates and response times after deploying.

## Code Examples

**Gradual rollout with feature flags — deploy once, then gradually enable for more users**

```ruby
# Feature flags decouple deploy from activation
if Flipper.enabled?(:new_pricing, current_user)
  # New pricing logic
else
  # Existing pricing logic
end

# Gradually roll out
Flipper.enable_percentage_of_actors(:new_pricing, 5)   # 5%
Flipper.enable_percentage_of_actors(:new_pricing, 25)  # 25%
Flipper.enable_percentage_of_actors(:new_pricing, 100) # Everyone
```


## Resources

- [Flipper Feature Flags](https://github.com/flippercloud/flipper) — Feature flag gem for Ruby — supports percentage rollouts, user targeting, and groups

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*