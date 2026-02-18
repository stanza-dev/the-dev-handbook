---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-server-sent-events"
---

# Server-Sent Events (SSE)

## Introduction

Need to push updates to clients in real-time? SSE provides a simple, unidirectional streaming protocol over HTTP. Unlike WebSockets, SSE works through standard HTTP, making it firewall-friendly and easier to implement for one-way communication.

## Key Concepts

- **SSE**: Server-Sent Events—one-way server-to-client streaming
- **Observable**: RxJS type for async event streams
- **MessageEvent**: Standard format for SSE messages
- **EventSource**: Browser API for consuming SSE

## Real World Context

SSE is perfect for:
- Live dashboards and metrics
- Progress updates for long-running tasks
- News feeds and notifications
- Stock price tickers

## Deep Dive

### Basic SSE Endpoint

```typescript
import { Sse, MessageEvent } from '@nestjs/common';
import { Observable, interval } from 'rxjs';
import { map } from 'rxjs/operators';

@Controller()
export class EventsController {
  @Sse('events')
  sendEvents(): Observable<MessageEvent> {
    return interval(1000).pipe(
      map(num => ({
        data: { count: num, timestamp: new Date().toISOString() },
      })),
    );
  }
}
```

### Custom Event Types

```typescript
@Sse('notifications')
sendNotifications(): Observable<MessageEvent> {
  return this.notificationService.getStream().pipe(
    map(notification => ({
      data: notification,
      type: notification.type, // 'message', 'alert', 'update'
      id: notification.id,
      retry: 30000, // Reconnect after 30s on disconnect
    })),
  );
}
```

### Client-Side Consumption

```javascript
const eventSource = new EventSource('/events');

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data);
};

eventSource.addEventListener('alert', (event) => {
  const data = JSON.parse(event.data);
  alert(data.message);
});

eventSource.onerror = () => {
  console.log('Connection lost, reconnecting...');
};
```

### Progress Updates

```typescript
@Injectable()
export class TaskService {
  private progressSubjects = new Map<string, Subject<number>>();

  startTask(taskId: string): void {
    const subject = new Subject<number>();
    this.progressSubjects.set(taskId, subject);
    
    // Simulate progress
    let progress = 0;
    const interval = setInterval(() => {
      progress += 10;
      subject.next(progress);
      if (progress >= 100) {
        clearInterval(interval);
        subject.complete();
      }
    }, 1000);
  }

  getProgress(taskId: string): Observable<MessageEvent> {
    const subject = this.progressSubjects.get(taskId);
    return subject.pipe(
      map(progress => ({ data: { taskId, progress } })),
    );
  }
}
```

## Common Pitfalls

1. **Not handling disconnects**: Clients may disconnect. Clean up resources.
2. **Memory leaks**: Subjects not completing can leak memory.
3. **Missing retry**: Set retry time for automatic reconnection.

## Best Practices

- Use SSE for one-way communication (server to client)
- Include event IDs for client-side deduplication
- Set appropriate retry intervals
- Complete observables when done to free resources

## Summary

SSE provides simple real-time server-to-client communication. Return an Observable<MessageEvent> from @Sse() endpoints. Use for progress updates, live feeds, and notifications. Remember to handle disconnects and complete streams.

## Resources

- [Server-Sent Events](https://docs.nestjs.com/techniques/server-sent-events) — Official SSE guide

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*