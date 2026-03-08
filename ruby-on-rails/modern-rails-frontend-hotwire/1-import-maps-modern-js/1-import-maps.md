---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-understanding-import-maps"
---

# Understanding Import Maps

## Introduction
Import Maps are the default JavaScript management approach in Rails 8, eliminating the need for Node.js, npm, or webpack. Instead of bundling JavaScript through a complex build pipeline, Import Maps let browsers resolve ES module specifiers directly to URLs.

## Key Concepts
- **Import Map**: A JSON mapping that tells the browser where to find JavaScript modules by name.
- **Pin**: A declaration in `config/importmap.rb` that maps a module name to a file or CDN URL.
- **ES Modules**: The native JavaScript module system using `import` and `export` statements.
- **Preloading**: Telling the browser to fetch a module early, before it is actually imported.

## Real World Context
In production Rails applications, Import Maps mean zero build step for JavaScript. When you deploy, there is no webpack compilation, no npm install, and no node_modules directory. This dramatically simplifies CI/CD pipelines and reduces deployment times. Every new Rails 8 application uses Import Maps by default.

## Deep Dive
When Rails generates a new application, it creates a `config/importmap.rb` file that defines all your JavaScript dependencies. This file produces a `<script type="importmap">` tag in your HTML.

Here is what the browser sees in the rendered HTML:

```html
<script type="importmap">
{
  "imports": {
    "application": "/assets/application-a1b2c3.js",
    "@hotwired/turbo-rails": "/assets/turbo.min-x7y8z9.js",
    "@hotwired/stimulus": "/assets/stimulus.min-d4e5f6.js"
  }
}
</script>
```

This mapping tells the browser that when any module calls `import "@hotwired/turbo-rails"`, it should fetch the file at the mapped URL. The fingerprinted filenames ensure proper cache busting.

The configuration that produces this lives in `config/importmap.rb`:

```ruby
pin "application", preload: true
pin "@hotwired/turbo-rails", to: "turbo.min.js", preload: true
pin "@hotwired/stimulus", to: "stimulus.min.js", preload: true
pin_all_from "app/javascript/controllers", under: "controllers"
```

Each `pin` statement maps a module name to a JavaScript file. The `preload: true` option adds a `<link rel="modulepreload">` tag so the browser starts downloading immediately.

To include Import Maps in your layout, Rails provides a single helper:

```erb
<head>
  <%= javascript_importmap_tags %>
</head>
```

This generates three things: the importmap script tag, modulepreload link tags, and a `<script type="module">` tag that loads your entry point.

## Common Pitfalls
1. **Assuming you need a build step** — Import Maps work without compilation. If you reach for webpack or esbuild in a new Rails 8 app, check if Import Maps already handle your use case.
2. **Using CommonJS syntax** — Import Maps only work with ES modules. Using `require()` or `module.exports` will not work. Always use `import` and `export`.

## Best Practices
1. **Preload critical modules** — Always set `preload: true` for your application entry point and core libraries like Turbo and Stimulus.
2. **Keep the entry point lean** — Your `application.js` should primarily import other modules rather than containing logic itself.

## Summary
- Import Maps are the default JavaScript approach in Rails 8, requiring no Node.js or bundler.
- `config/importmap.rb` maps module names to JavaScript files using `pin` statements.
- `javascript_importmap_tags` renders the import map, preload tags, and entry point.
- Browsers resolve `import` statements using the import map as ES modules.
- Preloading ensures critical modules are fetched early.

## Code Examples

**The default importmap.rb in Rails 8 — each pin maps a module name to a JavaScript file**

```ruby
# config/importmap.rb
pin "application", preload: true
pin "@hotwired/turbo-rails", to: "turbo.min.js", preload: true
pin "@hotwired/stimulus", to: "stimulus.min.js", preload: true
pin_all_from "app/javascript/controllers", under: "controllers"
```

**The application entry point uses standard ES module imports resolved by the import map**

```javascript
// app/javascript/application.js
import "@hotwired/turbo-rails"
import "controllers"
// The browser resolves these using the import map
```


## Resources

- [Working with JavaScript in Rails](https://guides.rubyonrails.org/working_with_javascript_in_rails.html) — Official Rails guide covering Import Maps, Turbo, and Stimulus
- [importmap-rails Repository](https://github.com/rails/importmap-rails) — Source code and README for the importmap-rails gem

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*