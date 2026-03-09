---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-alerting-incident-response"
---

# Alerting and Incident Response

## Introduction
Proactive alerting catches issues before users report them. Combined with a structured incident response process, alerting reduces downtime and prevents repeat incidents.

## Key Concepts
- **Health Check**: Endpoints that verify application and dependency health — liveness (process alive) and readiness (can serve traffic).
- **Alert Threshold**: A metric value that triggers a notification — e.g., error rate > 1% or p99 > 3 seconds.
- **Incident Runbook**: A step-by-step guide for responding to specific types of production incidents.

## Real World Context
At 2 AM, your error rate spikes to 15%. Without alerting, users experience errors for hours until someone checks in the morning. With PagerDuty integration, the on-call engineer is notified within minutes and follows the runbook to diagnose and fix the issue.

## Deep Dive

### Health Check Endpoints

```ruby
class HealthController < ApplicationController
  skip_before_action :authenticate_user!

  def live
    head :ok
  end

  def ready
    checks = {
      database: check_database,
      solid_queue: check_queue
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
    { status: 'ok' }
  rescue => e
    { status: 'error', message: e.message }
  end

  def check_queue
    SolidQueue::Job.count
    { status: 'ok' }
  rescue => e
    { status: 'error', message: e.message }
  end
end
```

### Alert Configuration

```ruby
class AlertService
  def self.critical(message, details = {})
    PagerDuty.trigger(
      routing_key: ENV['PAGERDUTY_KEY'],
      event_action: 'trigger',
      payload: {
        summary: message,
        severity: 'critical',
        custom_details: details
      }
    )
  end
end
```

### Automated Health Monitoring

```ruby
class SystemHealthCheckJob < ApplicationJob
  def perform
    error_rate = calculate_error_rate(5.minutes)
    AlertService.critical('High error rate', rate: error_rate) if error_rate > 0.05

    queue_latency = SolidQueue::ReadyExecution.maximum(:created_at)
    if queue_latency && queue_latency < 10.minutes.ago
      AlertService.critical('Queue backup', oldest_job: queue_latency)
    end
  end
end
```

### Incident Response Steps

1. **Acknowledge** — Confirm alert received
2. **Assess** — Check dashboards and logs
3. **Communicate** — Update status page
4. **Mitigate** — Apply quick fix (rollback, feature flag)
5. **Resolve** — Deploy permanent fix
6. **Review** — Post-incident analysis

## Common Pitfalls
1. **Too many alerts** — Alert fatigue causes engineers to ignore notifications. Only alert on actionable conditions.
2. **No runbook** — Without documented steps, incident response is slow and error-prone.

## Best Practices
1. **Alert on symptoms, not causes** — Alert on "error rate > 1%" not "database CPU > 80%". Symptoms directly impact users.
2. **Always do a post-incident review** — Identify root causes and prevent repeat incidents with systemic fixes.

## Summary
- Health checks verify liveness (process up) and readiness (dependencies connected).
- Alert on user-facing symptoms like error rates and response times.
- Follow a structured incident response: acknowledge, assess, communicate, mitigate, resolve, review.
- Write runbooks for common incident types to speed up response.

## Code Examples

**Readiness endpoint that checks database and Solid Queue — returns 503 if any dependency is down**

```ruby
# Readiness check — verifies database and job queue
def ready
  checks = {
    database: check_database,
    solid_queue: check_queue
  }
  all_ok = checks.values.all? { |c| c[:status] == 'ok' }
  render json: { status: all_ok ? 'healthy' : 'degraded', checks: checks },
         status: all_ok ? :ok : :service_unavailable
end
```


## Resources

- [Rails Application Monitoring Guide](https://guides.rubyonrails.org/active_support_instrumentation.html) — Official Rails guide on instrumentation and monitoring hooks

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*