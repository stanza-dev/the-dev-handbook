---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-legacy-ssr"
---

# Legacy SSR: renderToString and renderToStaticMarkup

These older APIs render React to a string synchronously. While still supported, they don't support streaming or Suspense.

## renderToString

Renders React to an HTML string that can be hydrated on the client:

```tsx
import { renderToString } from 'react-dom/server';

function handler(request, response) {
  const html = renderToString(<App />);
  
  response.send(`
    <!DOCTYPE html>
    <html>
      <body>
        <div id="root">${html}</div>
        <script src="/main.js"></script>
      </body>
    </html>
  `);
}
```

## renderToStaticMarkup

Similar to `renderToString`, but doesn't include React-specific attributes. The output cannot be hydrated:

```tsx
import { renderToStaticMarkup } from 'react-dom/server';

// Good for emails, static pages that don't need interactivity
const emailHtml = renderToStaticMarkup(
  <EmailTemplate user={user} />
);
```

## Comparison

| Feature | renderToString | renderToStaticMarkup |
|---------|----------------|----------------------|
| Hydration | ✅ Supported | ❌ Not supported |
| React attributes | ✅ Included | ❌ Excluded |
| Use case | Interactive apps | Static content |

## Why Avoid These APIs?

### 1. No Streaming

```tsx
// Blocks until EVERYTHING is ready
const html = renderToString(<App />); // Could take seconds!
```

### 2. No Suspense Support

```tsx
// Suspense boundaries throw errors or render fallbacks
function App() {
  return (
    <Suspense fallback={<Loading />}>
      <AsyncComponent /> {/* Will show Loading, not actual content */}
    </Suspense>
  );
}
```

### 3. Blocks the Server

The entire render must complete before sending any response, blocking other requests.

## When They're Still Useful

- **Email templates** - No JavaScript needed
- **PDF generation** - Static HTML output
- **Simple static pages** - No interactivity required
- **Legacy systems** - Gradual migration to streaming

## Migration Path

```tsx
// Before: Blocking
const html = renderToString(<App />);
res.send(html);

// After: Streaming
const { pipe } = renderToPipeableStream(<App />, {
  onShellReady() {
    pipe(res);
  }
});
```

## Resources

- [renderToString API Reference](https://react.dev/reference/react-dom/server/renderToString) — Official React documentation for renderToString

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*