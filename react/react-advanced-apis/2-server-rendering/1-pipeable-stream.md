---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-pipeable-stream"
---

# renderToPipeableStream: Node.js Streaming SSR

`renderToPipeableStream` renders a React tree to a Node.js Stream with full support for Suspense and streaming.

## Why Streaming?

Traditional SSR blocks until the entire page is ready. Streaming lets you:

1. **Send HTML progressively** - Users see content faster
2. **Integrate with Suspense** - Show loading states, stream in content as it's ready
3. **Improve Time to First Byte (TTFB)** - Start sending immediately

## Basic Usage

```tsx
import { renderToPipeableStream } from 'react-dom/server';
import App from './App';

function handler(request, response) {
  const { pipe, abort } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/main.js'],
    
    onShellReady() {
      // The shell (everything outside Suspense) is ready
      response.statusCode = 200;
      response.setHeader('Content-Type', 'text/html');
      pipe(response);
    },
    
    onShellError(error) {
      // Something went wrong rendering the shell
      response.statusCode = 500;
      response.send('<!doctype html><p>Error loading page</p>');
    },
    
    onAllReady() {
      // Everything including Suspense content is ready
      // Useful for crawlers/bots
    },
    
    onError(error) {
      console.error('Streaming error:', error);
    }
  });

  // Abort if the request is cancelled
  request.on('close', () => {
    abort();
  });
}
```

## Understanding the Callbacks

### onShellReady

Fires when the initial "shell" is ready - everything outside `<Suspense>` boundaries:

```tsx
function App() {
  return (
    <html>
      <head><title>My App</title></head>
      <body>
        <Header /> {/* Part of shell */}
        <Suspense fallback={<Spinner />}>
          <SlowContent /> {/* Streamed later */}
        </Suspense>
        <Footer /> {/* Part of shell */}
      </body>
    </html>
  );
}
```

### onAllReady

Fires when everything is ready, including all Suspense content. Use this for:

- **Crawlers/bots** that need complete HTML
- **Static generation** where you want the full page

```tsx
onAllReady() {
  // For bots, wait for everything
  if (isCrawler(request)) {
    response.statusCode = 200;
    pipe(response);
  }
}
```

## The HTML Output

Streaming produces HTML like this:

```html
<!DOCTYPE html>
<html>
  <head><title>My App</title></head>
  <body>
    <header>...</header>
    <!-- Suspense fallback initially -->
    <template id="B:0"></template>
    <div>Loading...</div>
    <!--/$-->
    <footer>...</footer>
    
    <!-- Later, streamed in: -->
    <div hidden id="S:0">Actual content!</div>
    <script>/* Swap fallback with content */</script>
  </body>
</html>
<script src="/main.js" async=""></script>
```

## Resources

- [renderToPipeableStream API Reference](https://react.dev/reference/react-dom/server/renderToPipeableStream) — Official React documentation for renderToPipeableStream

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*