---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-managing-dependencies"
---

# Managing Dependencies

Import Maps provides commands and conventions for managing JavaScript dependencies.

## Adding Packages

```bash
# Pin from CDN (jspm.io by default)
bin/importmap pin lodash

# Pin specific version
bin/importmap pin lodash@4.17.21

# Pin from unpkg CDN
bin/importmap pin react --from unpkg

# Pin from jsdelivr
bin/importmap pin vue --from jsdelivr
```

## Viewing Pinned Packages

```bash
# List all pinned packages
bin/importmap packages
```

## Updating Packages

```bash
# Update all packages
bin/importmap update

# Update specific package
bin/importmap update lodash
```

## Removing Packages

```bash
# Unpin a package
bin/importmap unpin lodash
```

## Vendoring Packages Locally

For packages you want to serve from your app:

```bash
# Download to vendor/javascript
bin/importmap vendor lodash
```

```ruby
# config/importmap.rb
pin 'lodash', to: 'vendor/lodash.js'
```

## Organizing Custom JavaScript

```ruby
# config/importmap.rb

# Single file
pin 'utils', to: 'utils.js'

# Directory of files
pin_all_from 'app/javascript/lib', under: 'lib'

# With subdirectories
pin_all_from 'app/javascript/components', under: 'components'
```

Usage:
```javascript
import { formatDate } from 'lib/dates'
import { Modal } from 'components/modal'
```

## Preloading for Performance

```ruby
# Preload critical packages
pin '@hotwired/turbo-rails', preload: true
pin '@hotwired/stimulus', preload: true

# Don't preload rarely-used packages
pin 'chart.js'  # Loaded on demand
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*