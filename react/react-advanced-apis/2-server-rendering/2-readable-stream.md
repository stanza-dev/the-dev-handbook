---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-readable-stream"
---

# renderToReadableStream: Web Streams API

`renderToReadableStream` renders a React tree to a Web Readable Stream. It's designed for modern edge runtimes like Cloudflare Workers, Deno, and Vercel Edge Functions.

## Web Streams vs Node Streams

| API | Environment | Stream Type |
|-----|-------------|-------------|
| `renderToPipeableStream` | Node.js | Node.js Stream |
| `renderToReadableStream` | Edge/Browser | Web ReadableStream |

## Basic Usage

```tsx
import { renderToReadableStream } from 'react-dom/server';
import App from './App';

async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js'],
    
    onError(error) {
      console.error('Streaming error:', error);
    }
  });

  return new Response(stream, {
    headers: { 'Content-Type': 'text/html' }
  });
}
```

## Waiting for All Content

Unlike `renderToPipeableStream`, this API returns a Promise. The stream starts immediately, but you can wait for all content:

```tsx
async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js']
  });

  // For crawlers, wait for everything
  if (isCrawler(request)) {
    await stream.allReady;
  }

  return new Response(stream, {
    headers: { 'Content-Type': 'text/html' }
  });
}
```

## Error Handling

Handle errors that occur before streaming starts:

```tsx
async function handler(request) {
  let statusCode = 200;
  
  try {
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      
      onError(error) {
        // Errors during streaming
        console.error(error);
        statusCode = 500;
      }
    });

    return new Response(stream, {
      status: statusCode,
      headers: { 'Content-Type': 'text/html' }
    });
  } catch (error) {
    // Error before streaming started (shell error)
    return new Response('<!doctype html><p>Error</p>', {
      status: 500,
      headers: { 'Content-Type': 'text/html' }
    });
  }
}
```

## Aborting the Stream

You can abort rendering using an AbortController:

```tsx
async function handler(request) {
  const controller = new AbortController();
  
  // Abort after 10 seconds
  setTimeout(() => controller.abort(), 10000);

  const stream = await renderToReadableStream(<App />, {
    signal: controller.signal,
    bootstrapScripts: ['/main.js']
  });

  return new Response(stream);
}
```

## Edge Runtime Example (Cloudflare Workers)

```tsx
export default {
  async fetch(request, env) {
    const stream = await renderToReadableStream(
      <App env={env} />,
      { bootstrapScripts: ['/client.js'] }
    );
    
    return new Response(stream, {
      headers: {
        'Content-Type': 'text/html',
        'Transfer-Encoding': 'chunked'
      }
    });
  }
};
```

## Resources

- [renderToReadableStream API Reference](https://react.dev/reference/react-dom/server/renderToReadableStream) — Official React documentation for renderToReadableStream

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*