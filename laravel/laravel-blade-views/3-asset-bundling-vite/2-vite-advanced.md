---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-vite-advanced"
---

# Advanced Vite Configuration

Master advanced Vite features for complex Laravel applications.

## Static Asset Handling

### Images and Fonts

Reference assets in CSS:

```css
/* resources/css/app.css */
.hero {
    background-image: url('../images/hero.jpg');
}

@font-face {
    font-family: 'CustomFont';
    src: url('../fonts/CustomFont.woff2') format('woff2');
}
```

Reference in JavaScript:

```javascript
// resources/js/app.js
import logoUrl from '../images/logo.svg';

document.getElementById('logo').src = logoUrl;
```

### Asset Helper in Blade

```blade
{{-- For images in public folder --}}
<img src="{{ asset('images/logo.png') }}" alt="Logo">

{{-- For Vite-processed assets (recommended) --}}
<img src="{{ Vite::asset('resources/images/logo.svg') }}" alt="Logo">
```

## Code Splitting

Vite automatically splits code for optimal loading:

```javascript
// Dynamic import for code splitting
const loadChart = async () => {
    const { Chart } = await import('chart.js');
    return Chart;
};

// Use when needed
document.getElementById('show-chart').addEventListener('click', async () => {
    const Chart = await loadChart();
    // Initialize chart
});
```

## Alias Configuration

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import path from 'path';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    resolve: {
        alias: {
            '@': path.resolve(__dirname, 'resources/js'),
            '@components': path.resolve(__dirname, 'resources/js/components'),
            '@utils': path.resolve(__dirname, 'resources/js/utils'),
        },
    },
});
```

Usage:

```javascript
// Instead of: import { helper } from '../../utils/helper'
import { helper } from '@utils/helper';
import Button from '@components/Button';
```

## React Integration

```bash
npm install react react-dom @vitejs/plugin-react
```

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.jsx'],
            refresh: true,
        }),
        react(),
    ],
});
```

```jsx
// resources/js/app.jsx
import '../css/app.css';
import { createRoot } from 'react-dom/client';
import App from './components/App';

const container = document.getElementById('app');
const root = createRoot(container);
root.render(<App />);
```

```blade
{{-- In Blade --}}
<!DOCTYPE html>
<html>
<head>
    @viteReactRefresh
    @vite(['resources/css/app.css', 'resources/js/app.jsx'])
</head>
<body>
    <div id="app"></div>
</body>
</html>
```

## Vue Integration

```bash
npm install vue @vitejs/plugin-vue
```

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        vue({
            template: {
                transformAssetUrls: {
                    base: null,
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

```javascript
// resources/js/app.js
import '../css/app.css';
import { createApp } from 'vue';
import App from './components/App.vue';

createApp(App).mount('#app');
```

## Environment Variables

Access environment variables in JavaScript:

```env
# .env
VITE_APP_NAME="My App"
VITE_API_URL="https://api.example.com"
```

```javascript
// Must be prefixed with VITE_
console.log(import.meta.env.VITE_APP_NAME);
console.log(import.meta.env.VITE_API_URL);

// Built-in variables
console.log(import.meta.env.MODE);  // 'development' or 'production'
console.log(import.meta.env.DEV);   // true in development
console.log(import.meta.env.PROD);  // true in production
```

## Custom Blade Refresh

```javascript
// vite.config.js
export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: [
                'resources/views/**',
                'routes/**',
                'app/View/Components/**',
            ],
        }),
    ],
});
```

## SSR Support

For server-side rendering:

```javascript
// vite.config.js
export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
            refresh: true,
        }),
    ],
});
```

## Subresource Integrity (SRI)

```php
// In AppServiceProvider boot()
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity');

// Or disable
Vite::useIntegrityKey(false);
```

## Prefetching

```php
// Prefetch assets for faster navigation
Vite::prefetch(concurrency: 3);
```

## Resources

- [Vite Configuration](https://vitejs.dev/config/) — Official Vite configuration reference

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*