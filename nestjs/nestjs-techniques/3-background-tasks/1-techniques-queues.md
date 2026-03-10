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

## Real World Context

Long-running operations block HTTP requests and cause timeouts. Sending a welcome email, generating a PDF report, or processing a video upload shouldn't keep the user waiting. Queues let you offload these tasks to background workers that process them asynchronously.

## Deep Dive

### Setup

Install the BullMQ integration and configure the Redis connection in your root module.

```bash
npm install @nestjs/bullmq bullmq
```

Register a named queue with `registerQueue()` after setting up the Redis connection with `forRoot()`.

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

Each named queue gets its own Redis key namespace, so you can register multiple queues independently.

### Producer (Adding Jobs)

Inject the queue with `@InjectQueue()` and call `add()` to enqueue jobs with options like delay and retries.

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

The `backoff` option with `exponential` type doubles the wait between each retry (1s, 2s, 4s).

### Consumer (Processing Jobs)

Extend `WorkerHost` and decorate the class with `@Processor()` to consume jobs from a named queue.

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

The `job.name` field lets you route different job types to different handlers within a single processor.

### Job Events

Use `@OnWorkerEvent()` to hook into job lifecycle events like completion and failure.

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

These event handlers are useful for logging, metrics collection, and triggering follow-up actions.

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

## Code Examples

**Setting up a Bull queue with BullModule and creating a processor to handle jobs**

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


## Resources

- [Queues](https://docs.nestjs.com/techniques/queues) — Official queues guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*