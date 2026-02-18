---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-vite-fundamentals"
---

# Introduction to Vite in Laravel

Vite is Laravel's default frontend build tool, replacing Laravel Mix. It offers blazing-fast development with Hot Module Replacement (HMR) and optimized production builds.

## Why Vite?

| Feature | Vite | Laravel Mix (Webpack) |
|---------|------|----------------------|
| **Dev Server Start** | ~300ms | ~10s |
| **Hot Reload** | Instant | 1-3s |
| **Build Time** | Fast (esbuild) | Slower (Webpack) |
| **Configuration** | Simple | Complex |
| **Native ESM** | Yes | No |

## Project Setup

New Laravel projects include Vite. Check your `package.json`:

```json
{
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "devDependencies": {
    "autoprefixer": "^10.4.0",
    "laravel-vite-plugin": "^1.0.0",
    "postcss": "^8.4.0",
    "tailwindcss": "^3.4.0",
    "vite": "^5.0.0"
  }
}
```

## Configuration

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
            ],
            refresh: true,  // Auto-refresh on Blade changes
        }),
    ],
});
```

## The @vite Directive

Include compiled assets in Blade:

```blade
<!DOCTYPE html>
<html>
<head>
    <title>My App</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body>
    {{ $slot }}
</body>
</html>
```

**In Development**, this generates:

```html
<script type="module" src="http://localhost:5173/@vite/client"></script>
<link rel="stylesheet" href="http://localhost:5173/resources/css/app.css">
<script type="module" src="http://localhost:5173/resources/js/app.js"></script>
```

**In Production** (after `npm run build`):

```html
<link rel="stylesheet" href="/build/assets/app-BrYLxZ9n.css">
<script type="module" src="/build/assets/app-DfJk2XMz.js"></script>
```

## Development Workflow

```bash
# Terminal 1: Start Laravel
php artisan serve

# Terminal 2: Start Vite dev server
npm run dev
```

Now your browser connects to Laravel at `localhost:8000`, and Vite serves assets from `localhost:5173` with HMR.

## CSS with Vite

### Tailwind CSS (Default)

```css
/* resources/css/app.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom styles */
.btn-primary {
    @apply px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700;
}
```

### PostCSS Configuration

```javascript
// postcss.config.js
export default {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```

### Importing CSS in JavaScript

```javascript
// resources/js/app.js
import '../css/app.css';

// Your JavaScript
console.log('App loaded!');
```

## JavaScript with Vite

### ES Modules

```javascript
// resources/js/app.js
import './bootstrap';
import { formatDate } from './utils/date';
import Alpine from 'alpinejs';

window.Alpine = Alpine;
Alpine.start();

console.log(formatDate(new Date()));
```

```javascript
// resources/js/utils/date.js
export function formatDate(date) {
    return new Intl.DateTimeFormat('en-US', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
    }).format(date);
}
```

### Installing NPM Packages

```bash
npm install alpinejs
npm install axios
npm install lodash-es  # ES module version
```

```javascript
// resources/js/app.js
import Alpine from 'alpinejs';
import axios from 'axios';
import { debounce } from 'lodash-es';

window.Alpine = Alpine;
window.axios = axios;
window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';

Alpine.start();
```

## Multiple Entry Points

```javascript
// vite.config.js
export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
                'resources/css/admin.css',
                'resources/js/admin.js',
            ],
            refresh: true,
        }),
    ],
});
```

```blade
{{-- Main app layout --}}
@vite(['resources/css/app.css', 'resources/js/app.js'])

{{-- Admin layout --}}
@vite(['resources/css/admin.css', 'resources/js/admin.js'])
```

## Production Build

```bash
npm run build
```

This creates optimized assets in `public/build/`:
- Minified CSS and JavaScript
- Content-hashed filenames for cache busting
- Manifest file for Laravel to resolve paths

```
public/build/
├── assets/
│   ├── app-BrYLxZ9n.css
│   └── app-DfJk2XMz.js
└── manifest.json
```

## Troubleshooting

```blade
{{-- Check if Vite is running --}}
@if (app()->environment('local'))
    <p>Make sure to run: <code>npm run dev</code></p>
@endif
```

Common issues:
1. **Blank page**: Vite dev server not running
2. **CORS errors**: Check Vite server URL in config
3. **Assets not found in production**: Run `npm run build`

## Resources

- [Laravel Vite](https://laravel.com/docs/12.x/vite) — Official Laravel Vite documentation

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*