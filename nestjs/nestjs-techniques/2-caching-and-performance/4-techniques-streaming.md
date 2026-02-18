---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-streaming"
---

# Streaming Responses

## Introduction

Large files shouldn't be loaded entirely into memory. Streaming sends data in chunks as it becomes available, reducing memory usage and enabling real-time data delivery.

## Key Concepts

- **StreamableFile**: NestJS class for file streaming
- **Readable Stream**: Node.js stream for chunked data
- **Server-Sent Events**: One-way streaming to clients
- **Content-Disposition**: Header for file downloads

## Deep Dive

### File Download

```typescript
import { StreamableFile } from '@nestjs/common';
import { createReadStream } from 'fs';

@Get('download/:filename')
getFile(
  @Param('filename') filename: string,
  @Res({ passthrough: true }) res: Response,
): StreamableFile {
  const file = createReadStream(\`./uploads/\${filename}\`);
  
  res.set({
    'Content-Type': 'application/pdf',
    'Content-Disposition': \`attachment; filename="\${filename}"\`,
  });
  
  return new StreamableFile(file);
}
```

### Server-Sent Events

```typescript
import { Sse, MessageEvent } from '@nestjs/common';
import { Observable, interval } from 'rxjs';
import { map } from 'rxjs/operators';

@Sse('events')
sendEvents(): Observable<MessageEvent> {
  return interval(1000).pipe(
    map(num => ({
      data: { timestamp: new Date().toISOString(), count: num },
    })),
  );
}
```

### Streaming from Database

```typescript
@Get('export')
async exportData(@Res() res: Response) {
  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Disposition', 'attachment; filename="export.csv"');
  
  const stream = this.dataService.createExportStream();
  stream.pipe(res);
}
```

## Common Pitfalls

1. **Memory leaks**: Not properly closing streams can leak resources.
2. **Error handling**: Stream errors need explicit handling.
3. **Backpressure**: Fast producers can overwhelm slow consumers.

## Best Practices

- Use StreamableFile for file downloads
- Handle stream errors and cleanup
- Set appropriate Content-Type and Content-Disposition headers
- Use SSE for real-time updates (simpler than WebSockets for one-way)

## Summary

Streaming sends data in chunks rather than buffering entirely in memory. Use StreamableFile for file downloads, SSE for real-time events, and handle stream errors properly. Set Content-Disposition for downloads.

## Resources

- [Streaming Files](https://docs.nestjs.com/techniques/streaming-files) — Official streaming guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*