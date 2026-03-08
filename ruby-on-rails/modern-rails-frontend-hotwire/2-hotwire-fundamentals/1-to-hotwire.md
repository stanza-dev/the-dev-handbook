---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-introduction-to-hotwire"
---

# What Is Hotwire

## Introduction
Hotwire stands for HTML Over The Wire. It is a collection of techniques for building interactive web applications by sending HTML from the server instead of JSON. Hotwire is the default frontend approach in Rails 8.

## Key Concepts
- **HTML Over The Wire**: The server sends rendered HTML fragments instead of JSON, and the client updates the DOM directly.
- **Turbo**: The core library handling navigation, partial updates, and real-time streaming without writing JavaScript.
- **Stimulus**: A lightweight JavaScript framework for adding behavior to server-rendered HTML.
- **Progressive Enhancement**: The application works with basic HTML and gets enhanced with Turbo and Stimulus.

## Real World Context
Traditional SPAs require a separate frontend in React/Vue, a JSON API, and client-side state management. Hotwire eliminates this entire layer. Companies like Basecamp, HEY, and Shopify use Hotwire to build complex applications with a fraction of the JavaScript.

## Deep Dive
### The Hotwire Stack

```
Hotwire
├── Turbo
│   ├── Turbo Drive    — Fast navigation (replaces full page reloads)
│   ├── Turbo Frames   — Partial page updates (independent sections)
│   └── Turbo Streams  — Real-time updates (append, replace, remove, morph)
└── Stimulus           — JavaScript sprinkles (behavior on HTML elements)
```

Each component solves a specific problem. Turbo Drive handles page navigation. Turbo Frames update portions independently. Turbo Streams push real-time changes over WebSockets. Stimulus adds targeted JavaScript behavior.

### Setup in Rails 8

Hotwire is included by default. The JavaScript setup is two imports:

```javascript
// app/javascript/application.js
import "@hotwired/turbo-rails"
import "controllers"
```

The first activates Turbo Drive, Frames, and Streams. The second loads Stimulus controllers. Every link click and form submission is automatically intercepted by Turbo Drive.

### What You Don't Need

With Hotwire you typically skip: a separate JSON API, a JavaScript state library, a client-side router, SSR setup for a JS framework, and a separate frontend build pipeline. Rails views are your frontend.

## Common Pitfalls
1. **Thinking Hotwire means no JavaScript** — Stimulus controllers are still JavaScript, but small and focused instead of a full SPA.
2. **Trying to build SPA patterns with Hotwire** — Hotwire works best when you lean into server rendering.

## Best Practices
1. **Start with Turbo Drive and add complexity only when needed** — Try Frames before Streams, Streams before custom Stimulus.
2. **Keep the server in charge** — Let Rails controllers and views decide what to render.

## Summary
- Hotwire sends HTML from the server, eliminating the need for a JavaScript SPA.
- The stack: Turbo (Drive, Frames, Streams) for HTML delivery + Stimulus for behavior.
- Rails 8 includes Hotwire by default with two import statements.
- No need for JSON APIs, client-side routing, or state management libraries.

## Code Examples

**The complete Hotwire JavaScript setup in Rails 8 — two imports activate the entire stack**

```javascript
// app/javascript/application.js
import "@hotwired/turbo-rails"  // Turbo Drive, Frames, Streams
import "controllers"            // All Stimulus controllers
// No framework init, no store, no router config.
// Turbo intercepts all links and forms automatically.
```


## Resources

- [Hotwire Official Site](https://hotwired.dev/) — Official Hotwire documentation for Turbo and Stimulus
- [Working with JavaScript in Rails](https://guides.rubyonrails.org/working_with_javascript_in_rails.html) — Rails guide covering the Hotwire integration

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*