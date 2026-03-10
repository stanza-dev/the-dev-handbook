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

## Real World Context

Large files—reports, exports, media—can't be loaded entirely into memory. A CSV export with 100,000 rows would consume gigabytes of RAM if buffered. Streaming sends data in chunks, keeping memory usage constant regardless of file size.

## Deep Dive

### File Download

Use `StreamableFile` with `createReadStream()` to send files without buffering them entirely in memory.

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

The `passthrough: true` option is required so NestJS can still set status codes and headers alongside the stream.

### Server-Sent Events

Return an `Observable<MessageEvent>` from a method decorated with `@Sse()` to push real-time updates to clients.

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

Clients connect with the browser's `EventSource` API and receive each emitted object as a text event.

### Streaming from Database

For large exports, pipe a database cursor stream directly to the response to keep memory usage constant.

```typescript
@Get('export')
async exportData(@Res() res: Response) {
  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Disposition', 'attachment; filename="export.csv"');
  
  const stream = this.dataService.createExportStream();
  stream.pipe(res);
}
```

Using `@Res()` without `passthrough` gives you full control over the response but bypasses NestJS interceptors.

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

## Code Examples

**Using StreamableFile to stream file data as a response with proper headers**

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


## Resources

- [Streaming Files](https://docs.nestjs.com/techniques/streaming-files) — Official streaming guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*