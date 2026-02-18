---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-alerting-incident-response"
---

# Alerting Incident Response

Proactive alerting catches issues before users report them.

## Health Check Endpoints

```ruby
# app/controllers/health_controller.rb
class HealthController < ApplicationController
  skip_before_action :authenticate_user!
  
  # Basic liveness check
  def live
    head :ok
  end
  
  # Detailed readiness check
  def ready
    checks = {
      database: check_database,
      redis: check_redis,
      sidekiq: check_sidekiq,
      disk_space: check_disk_space
    }
    
    all_healthy = checks.values.all? { |c| c[:status] == 'ok' }
    
    render json: {
      status: all_healthy ? 'healthy' : 'degraded',
      checks: checks,
      timestamp: Time.current.iso8601
    }, status: all_healthy ? :ok : :service_unavailable
  end
  
  private
  
  def check_database
    ActiveRecord::Base.connection.execute('SELECT 1')
    { status: 'ok', latency: measure { ActiveRecord::Base.connection.execute('SELECT 1') } }
  rescue => e
    { status: 'error', message: e.message }
  end
  
  def check_redis
    latency = measure { Redis.current.ping }
    { status: 'ok', latency: latency }
  rescue => e
    { status: 'error', message: e.message }
  end
  
  def check_sidekiq
    stats = Sidekiq::Stats.new
    latency = Sidekiq::Queue.new.latency
    
    {
      status: latency < 300 ? 'ok' : 'warning',
      queue_latency: latency,
      enqueued: stats.enqueued
    }
  end
  
  def measure
    start = Process.clock_gettime(Process::CLOCK_MONOTONIC)
    yield
    ((Process.clock_gettime(Process::CLOCK_MONOTONIC) - start) * 1000).round(2)
  end
end
```

## Alert Configuration

```ruby
# config/initializers/alerts.rb
class AlertService
  def self.critical(message, details = {})
    # PagerDuty integration
    PagerDuty.trigger(
      routing_key: ENV['PAGERDUTY_KEY'],
      event_action: 'trigger',
      payload: {
        summary: message,
        severity: 'critical',
        custom_details: details
      }
    )
    
    # Slack notification
    Slack.notify(
      channel: '#incidents',
      text: "🚨 CRITICAL: #{message}",
      attachments: [{ fields: details.map { |k, v| { title: k, value: v } } }]
    )
  end
end
```

## Automated Alerting Rules

```ruby
# app/jobs/system_health_check_job.rb
class SystemHealthCheckJob < ApplicationJob
  def perform
    # Check error rate
    error_rate = calculate_error_rate(5.minutes)
    if error_rate > 0.05
      AlertService.critical('High error rate', rate: "#{(error_rate * 100).round(1)}%")
    end
    
    # Check queue latency
    Sidekiq::Queue.all.each do |queue|
      if queue.latency > 600
        AlertService.critical('Queue backup', queue: queue.name, latency: queue.latency)
      end
    end
    
    # Check response time
    p99_response = calculate_p99_response_time(5.minutes)
    if p99_response > 2000
      AlertService.critical('Slow response times', p99: "#{p99_response}ms")
    end
  end
end
```

## Incident Response Runbook

1. **Acknowledge**: Confirm alert received
2. **Assess**: Check dashboards and logs
3. **Communicate**: Update status page
4. **Mitigate**: Apply quick fix if possible
5. **Resolve**: Deploy permanent fix
6. **Review**: Post-incident analysis

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*