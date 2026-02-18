---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-queues"
---

# Queues (BullMQ)

## Introduction

Some operations are too slow for synchronous handling—sending emails, processing images, generating reports. Queues offload these tasks to background workers, keeping your API responsive while work happens asynchronously.

## Key Concepts

- **Queue**: FIFO data structure for jobs
- **Producer**: Adds jobs to the queue
- **Consumer/Processor**: Handles jobs from the queue
- **BullMQ**: High-performance Redis-based queue library

## Deep Dive

### Setup

```bash
npm install @nestjs/bullmq bullmq
```

```typescript
import { BullModule } from '@nestjs/bullmq';

@Module({
  imports: [
    BullModule.forRoot({
      connection: {
        host: 'localhost',
        port: 6379,
      },
    }),
    BullModule.registerQueue({
      name: 'email',
    }),
  ],
})
export class AppModule {}
```

### Producer (Adding Jobs)

```typescript
import { InjectQueue } from '@nestjs/bullmq';
import { Queue } from 'bullmq';

@Injectable()
export class EmailService {
  constructor(@InjectQueue('email') private emailQueue: Queue) {}

  async sendWelcomeEmail(userId: string) {
    await this.emailQueue.add('welcome', { userId }, {
      delay: 5000, // Wait 5 seconds
      attempts: 3, // Retry 3 times on failure
      backoff: { type: 'exponential', delay: 1000 },
    });
  }
}
```

### Consumer (Processing Jobs)

```typescript
import { Processor, WorkerHost } from '@nestjs/bullmq';
import { Job } from 'bullmq';

@Processor('email')
export class EmailProcessor extends WorkerHost {
  async process(job: Job<{ userId: string }>) {
    console.log(\`Processing job \${job.id} for user \${job.data.userId}\`);
    
    switch (job.name) {
      case 'welcome':
        await this.sendWelcomeEmail(job.data.userId);
        break;
    }
  }

  private async sendWelcomeEmail(userId: string) {
    // Send email logic
  }
}
```

### Job Events

```typescript
@Processor('email')
export class EmailProcessor extends WorkerHost {
  @OnWorkerEvent('completed')
  onCompleted(job: Job) {
    console.log(\`Job \${job.id} completed\`);
  }

  @OnWorkerEvent('failed')
  onFailed(job: Job, error: Error) {
    console.error(\`Job \${job.id} failed: \${error.message}\`);
  }
}
```

## Common Pitfalls

1. **No Redis**: BullMQ requires Redis. Configure connection properly.
2. **Job data not serializable**: Job data must be JSON serializable.
3. **Blocking processors**: Don't block—use async operations.

## Best Practices

- Use meaningful queue and job names
- Configure retry policies with exponential backoff
- Monitor queue health and failed jobs
- Use separate queues for different job types

## Summary

BullMQ provides Redis-backed queues for background job processing. Producers add jobs with @InjectQueue(), consumers process them with @Processor(). Configure retries and delays for robust job handling.

## Resources

- [Queues](https://docs.nestjs.com/techniques/queues) — Official queues guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*