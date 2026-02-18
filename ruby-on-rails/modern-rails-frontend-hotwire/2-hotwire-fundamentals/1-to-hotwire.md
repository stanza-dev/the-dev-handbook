---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-introduction-to-hotwire"
---

# Introduction To Hotwire

Hotwire is HTML Over The Wire - a set of techniques for building modern web applications with minimal JavaScript.

## The Hotwire Philosophy

Traditional SPAs send JSON and render on the client. Hotwire sends HTML and updates the DOM directly:

```
Traditional SPA:
Server -> JSON -> JavaScript renders HTML -> DOM update

Hotwire:
Server -> HTML -> Direct DOM update
```

## Hotwire Components

1. **Turbo Drive**: Intercepts links/forms, fetches pages via AJAX, swaps content
2. **Turbo Frames**: Update portions of the page independently
3. **Turbo Streams**: Deliver real-time updates over WebSocket
4. **Stimulus**: Add JavaScript behavior to HTML elements

## Setup in Rails 7+

Hotwire is included by default:

```ruby
# Gemfile (already included)
gem 'turbo-rails'
gem 'stimulus-rails'
```

```ruby
# config/importmap.rb
pin '@hotwired/turbo-rails', to: 'turbo.min.js', preload: true
pin '@hotwired/stimulus', to: 'stimulus.min.js', preload: true
pin_all_from 'app/javascript/controllers', under: 'controllers'
```

```javascript
// app/javascript/application.js
import '@hotwired/turbo-rails'
import 'controllers'
```

## Benefits Over SPAs

- **Less JavaScript**: Most features need zero client-side code
- **Server rendering**: SEO-friendly, fast initial load
- **Progressive enhancement**: Works without JavaScript
- **Rails conventions**: Leverage existing knowledge
- **Simpler architecture**: No client state management

See [Hotwire Documentation](https://hotwired.dev/) and [Working with JavaScript in Rails](https://guides.rubyonrails.org/working_with_javascript_in_rails.html).

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*