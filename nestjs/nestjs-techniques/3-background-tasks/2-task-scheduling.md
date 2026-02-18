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

## Deep Dive

### Setup

```bash
npm install @nestjs/schedule
```

```typescript
import { ScheduleModule } from '@nestjs/schedule';

@Module({
  imports: [ScheduleModule.forRoot()],
})
export class AppModule {}
```

### Cron Jobs

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

### Intervals and Timeouts

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

### Dynamic Scheduling

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

NestJS scheduling provides @Cron, @Interval, and @Timeout decorators for scheduled tasks. Use SchedulerRegistry for dynamic job management. Keep jobs fast and handle multiple instances with distributed locks.

## Resources

- [Task Scheduling](https://docs.nestjs.com/techniques/task-scheduling) — Official scheduling guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*