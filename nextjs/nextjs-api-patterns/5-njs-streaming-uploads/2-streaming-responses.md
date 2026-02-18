---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-streaming-responses"
---

# Streaming Responses

## Introduction

Some responses take time—AI completions, large data exports, real-time updates. Instead of making users wait for the entire response, stream data as it becomes available.

## Key Concepts

**Streaming** sends data in chunks:

- **ReadableStream**: Web API for streaming data
- **Server-Sent Events (SSE)**: One-way real-time updates
- **Text Streaming**: Character-by-character for AI responses

## Real World Context

Streaming enables:
- ChatGPT-like typing effects for AI responses
- Real-time notifications and feeds
- Progress updates for long-running tasks
- Large file downloads without memory issues

## Deep Dive

### Basic ReadableStream

```typescript
export async function GET() {
  const encoder = new TextEncoder();

  const stream = new ReadableStream({
    async start(controller) {
      for (let i = 0; i < 10; i++) {
        const data = JSON.stringify({ count: i, time: Date.now() });
        controller.enqueue(encoder.encode(`data: ${data}\n\n`));
        await new Promise(resolve => setTimeout(resolve, 500));
      }
      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    },
  });
}
```

### AI Streaming Response

```typescript
export async function POST(request: Request) {
  const { prompt } = await request.json();

  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      model: 'gpt-4',
      messages: [{ role: 'user', content: prompt }],
      stream: true,
    }),
  });

  // Forward the stream directly
  return new Response(response.body, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
    },
  });
}
```

### Client-Side Consumption

```typescript
// Using EventSource (for GET endpoints)
'use client';

function Notifications() {
  const [messages, setMessages] = useState<string[]>([]);

  useEffect(() => {
    const eventSource = new EventSource('/api/notifications');
    
    eventSource.onmessage = (event) => {
      setMessages(prev => [...prev, event.data]);
    };
    
    eventSource.onerror = () => {
      eventSource.close();
    };
    
    return () => eventSource.close();
  }, []);

  return (
    <ul>
      {messages.map((msg, i) => <li key={i}>{msg}</li>)}
    </ul>
  );
}
```

```typescript
// Using fetch for POST streams
async function streamChat(prompt: string) {
  const response = await fetch('/api/chat', {
    method: 'POST',
    body: JSON.stringify({ prompt }),
  });

  const reader = response.body?.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader!.read();
    if (done) break;
    
    const text = decoder.decode(value);
    console.log('Received chunk:', text);
  }
}
```

## Common Pitfalls

1. **Forgetting headers**: SSE requires `Content-Type: text/event-stream` and `Cache-Control: no-cache`.

2. **Not closing streams**: Always call `controller.close()` when done or handle errors.

3. **EventSource only works with GET**: For POST requests, use fetch with stream reading.

## Best Practices

- **Use SSE for simple real-time updates**: Simpler than WebSockets for one-way data
- **Handle disconnections**: Implement reconnection logic on the client
- **Set reasonable timeouts**: Don't keep connections open forever
- **Forward streams directly when possible**: Avoid buffering entire responses

## Summary

Streaming sends data to clients as it becomes available, perfect for AI responses, real-time updates, and large data transfers. Use ReadableStream with SSE format, set the correct headers, and consume with EventSource (GET) or fetch stream reading (POST).

## Resources

- [Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) — MDN Streams API reference

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*