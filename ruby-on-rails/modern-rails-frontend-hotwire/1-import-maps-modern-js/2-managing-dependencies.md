---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-managing-dependencies"
---

# Managing Dependencies with Import Maps

## Introduction
Import Maps provides a CLI tool for adding, updating, and removing JavaScript packages without npm or yarn. You can pull packages from CDNs or vendor them locally for full control.

## Key Concepts
- **Pinning**: Adding a JavaScript package to your import map so it can be imported by name.
- **CDN Source**: Where the package is downloaded from — jspm.io (default), unpkg, or jsdelivr.
- **Vendoring**: Downloading a package into `vendor/javascript` so it is served from your own server.
- **`pin_all_from`**: Maps every file in a directory to an importable module under a namespace.

## Real World Context
In a typical Rails 8 project, you might need a date library, charting library, or utility like lodash. With Import Maps, adding any of these takes a single command — no package.json, no lock file, no node_modules.

## Deep Dive
### Adding Packages

The `bin/importmap` CLI handles package management:

```bash
bin/importmap pin lodash-es
bin/importmap pin sortablejs@1.15.0
bin/importmap pin chart.js --from jsdelivr
```

Each command adds a line to `config/importmap.rb` with the resolved CDN URL.

### Updating and Removing

```bash
bin/importmap packages    # List all pinned packages
bin/importmap update      # Update all packages
bin/importmap unpin lodash-es  # Remove a package
```

The `packages` command shows current versions and source URLs for auditing.

### Vendoring Locally

For CDN-free production deployments, vendor packages:

```bash
bin/importmap vendor sortablejs
```

This downloads to `vendor/javascript/` and updates the pin to reference the local copy:

```ruby
pin "sortablejs", to: "vendor/sortablejs.js"
```

Vendored packages are served through the Rails asset pipeline.

### Organizing Custom JavaScript

Use `pin` for single files and `pin_all_from` for directories:

```ruby
pin "utils/dates", to: "utils/dates.js"
pin_all_from "app/javascript/services", under: "services"
```

With `pin_all_from`, every `.js` file in the directory becomes importable under the namespace.

## Common Pitfalls
1. **Using CommonJS packages** — Import Maps require ES modules. Use ESM variants like `lodash-es` instead of `lodash`.
2. **Forgetting to vendor before offline deploy** — If production servers cannot reach CDNs, vendor all external packages.

## Best Practices
1. **Preload selectively** — Only preload packages needed on every page. Charting libraries used on one page should load on demand.
2. **Vendor for production stability** — Vendoring eliminates CDN dependency from deployments.

## Summary
- Use `bin/importmap pin` to add packages and `bin/importmap unpin` to remove them.
- Use `bin/importmap vendor` to download packages locally.
- `pin_all_from` maps a directory of files to importable modules.
- Always use ESM-compatible packages.
- Preload only critical packages.

## Code Examples

**A real-world importmap.rb with CDN pins, vendored packages, and local code directories**

```ruby
# config/importmap.rb — mixing CDN, vendored, and local code
pin "application", preload: true
pin "@hotwired/turbo-rails", to: "turbo.min.js", preload: true
pin "sortablejs", to: "https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/modular/sortable.esm.js"
pin "chart.js", to: "vendor/chart.js"
pin_all_from "app/javascript/controllers", under: "controllers"
pin_all_from "app/javascript/helpers", under: "helpers"
```


## Resources

- [importmap-rails Usage](https://github.com/rails/importmap-rails#usage) — Detailed usage for pinning, vendoring, and managing packages

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*