---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-sidekiq-setup"
---

# Sidekiq Setup

Sidekiq is the most popular and performant job backend for Rails.

## Installation

```ruby
# Gemfile
gem 'sidekiq'
```

```ruby
# config/application.rb
config.active_job.queue_adapter = :sidekiq
```

## Configuration

```yaml
# config/sidekiq.yml
:concurrency: 10
:queues:
  - [critical, 3]
  - [default, 2]
  - [low, 1]

production:
  :concurrency: 25

development:
  :concurrency: 5
```

## Redis Configuration

```ruby
# config/initializers/sidekiq.rb
Sidekiq.configure_server do |config|
  config.redis = {
    url: ENV.fetch('REDIS_URL', 'redis://localhost:6379/0'),
    ssl_params: { verify_mode: OpenSSL::SSL::VERIFY_NONE }
  }
end

Sidekiq.configure_client do |config|
  config.redis = {
    url: ENV.fetch('REDIS_URL', 'redis://localhost:6379/0')
  }
end
```

## Running Sidekiq

```bash
# Start worker process
bundle exec sidekiq

# With specific config
bundle exec sidekiq -C config/sidekiq.yml

# Specific queues only
bundle exec sidekiq -q critical -q default
```

## Web UI

```ruby
# config/routes.rb
require 'sidekiq/web'

Rails.application.routes.draw do
  # Protect with authentication
  authenticate :user, ->(u) { u.admin? } do
    mount Sidekiq::Web => '/sidekiq'
  end
end
```

## Environment Variables

```bash
# .env
REDIS_URL=redis://localhost:6379/0
SIDEKIQ_CONCURRENCY=10
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*