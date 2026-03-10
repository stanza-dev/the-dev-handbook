---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-task-scheduling"
---

# Task Scheduling

## Introduction

Some tasks need to run on a schedule—database cleanup, report generation, cache refresh. NestJS's scheduling module provides cron-based scheduling with a declarative, decorator-based API.

## Key Concepts

- **@Cron()**: Schedule method execution with cron expression
- **@Interval()**: Execute every N milliseconds
- **@Timeout()**: Execute once after N milliseconds
- **Cron Expression**: String defining schedule (second minute hour day month weekday)

## Real World Context

Many applications need recurring tasks: clearing expired sessions, sending daily digest emails, syncing data from external APIs, generating nightly reports. The @nestjs/schedule package provides cron-based and interval-based scheduling without external tools like crontab.

## Deep Dive

### Setup

Install the schedule package and register `ScheduleModule` in your root module.

```bash
npm install @nestjs/schedule
```

Calling `forRoot()` starts the scheduler and scans for decorated methods.

```typescript
import { ScheduleModule } from '@nestjs/schedule';

@Module({
  imports: [ScheduleModule.forRoot()],
})
export class AppModule {}
```

Once registered, any injectable class can use `@Cron()`, `@Interval()`, or `@Timeout()` decorators.

### Cron Jobs

Decorate methods with `@Cron()` using a cron expression or a `CronExpression` constant for readable schedules.

```typescript
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class TasksService {
  @Cron('45 * * * * *')
  handleCron() {
    console.log('Called when second is 45');
  }

  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  handleMidnight() {
    console.log('Running daily cleanup');
  }

  @Cron('0 0 * * 0') // Every Sunday at midnight
  weeklyReport() {
    this.reportService.generateWeeklyReport();
  }
}
```

NestJS cron expressions use six fields (second, minute, hour, day, month, weekday), unlike the standard five-field format.

### Intervals and Timeouts

Use `@Interval()` for recurring execution at fixed millisecond intervals, or `@Timeout()` for a one-time delayed execution.

```typescript
@Injectable()
export class TasksService {
  @Interval(10000) // Every 10 seconds
  handleInterval() {
    console.log('Called every 10 seconds');
  }

  @Timeout(5000) // Once after 5 seconds
  handleTimeout() {
    console.log('Called once after 5 seconds');
  }
}
```

`@Timeout()` fires exactly once after the delay elapses, which is useful for deferred initialization tasks.

### Dynamic Scheduling

Inject `SchedulerRegistry` to create, start, stop, and remove cron jobs at runtime.

```typescript
import { SchedulerRegistry } from '@nestjs/schedule';
import { CronJob } from 'cron';

@Injectable()
export class DynamicTasksService {
  constructor(private schedulerRegistry: SchedulerRegistry) {}

  addCronJob(name: string, cronTime: string) {
    const job = new CronJob(cronTime, () => {
      console.log(\`Running \${name}\`);
    });

    this.schedulerRegistry.addCronJob(name, job);
    job.start();
  }

  deleteCronJob(name: string) {
    this.schedulerRegistry.deleteCronJob(name);
  }
}
```

Dynamic jobs are registered by name, so you can look them up, stop, or delete them later via the registry.

### Cron Expression Reference

| Expression | Schedule |
|-----------|----------|
| `* * * * * *` | Every second |
| `0 * * * * *` | Every minute |
| `0 0 * * * *` | Every hour |
| `0 0 0 * * *` | Every day at midnight |
| `0 0 0 * * 0` | Every Sunday at midnight |

## Common Pitfalls

1. **Multiple instances**: In clustered apps, all instances run the job. Use distributed locks.
2. **Long-running jobs**: Don't block—jobs should be quick or offloaded to queues.
3. **Timezone confusion**: Cron runs in server timezone. Be explicit about times.

## Best Practices

- Use CronExpression constants for readability
- Add logging to track job execution
- Use queues for long-running jobs
- Handle errors gracefully—don't crash on failure

## Summary

- @Cron(), @Interval(), and @Timeout() decorators handle scheduled tasks declaratively
- Use SchedulerRegistry for dynamic job creation and management at runtime
- Use CronExpression constants for readable cron schedules
- Keep scheduled jobs fast; offload heavy work to queues
- Use distributed locks to prevent duplicate execution in clustered deployments

## Code Examples

**Importing and registering the ScheduleModule in the app module**

```typescript
import { ScheduleModule } from '@nestjs/schedule';

@Module({
  imports: [ScheduleModule.forRoot()],
})
export class AppModule {}
```


## Resources

- [Task Scheduling](https://docs.nestjs.com/techniques/task-scheduling) — Official scheduling guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*