---
source_course: "react-performance"
source_lesson: "react-performance-bundle-analysis"
---

# Bundle Analysis

Understand and optimize your bundle size.

## Analyzing Bundle Size

### Vite

```bash
npm install -D rollup-plugin-visualizer
```

```js
// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

export default {
  plugins: [
    visualizer({
      filename: 'stats.html',
      open: true,
      gzipSize: true,
    }),
  ],
};
```

### Create React App

```bash
npm install -D source-map-explorer
```

```json
// package.json
{
  "scripts": {
    "analyze": "source-map-explorer 'build/static/js/*.js'"
  }
}
```

### Next.js

```bash
npm install -D @next/bundle-analyzer
```

```js
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({});
```

## Common Bundle Bloat

### 1. Moment.js Locale Files

```js
// webpack.config.js
const webpack = require('webpack');

module.exports = {
  plugins: [
    new webpack.IgnorePlugin({
      resourceRegExp: /^\.\/locale$/,
      contextRegExp: /moment$/,
    }),
  ],
};

// Or just use date-fns or dayjs instead!
```

### 2. Full lodash Import

```jsx
// ❌ Imports entire lodash
import { debounce } from 'lodash';

// ✅ Tree-shakeable
import debounce from 'lodash/debounce';

// ✅ Or use lodash-es
import { debounce } from 'lodash-es';
```

### 3. Icon Libraries

```jsx
// ❌ Imports all icons
import { FaHome } from 'react-icons/fa';

// ✅ Direct import
import FaHome from 'react-icons/fa/FaHome';
```

## Dynamic Imports for Heavy Libraries

```jsx
// Heavy charting library
function ChartComponent({ data }) {
  const [Chart, setChart] = useState(null);
  
  useEffect(() => {
    import('recharts').then(module => {
      setChart(() => module.LineChart);
    });
  }, []);
  
  if (!Chart) return <Skeleton />;
  
  return (
    <Chart data={data}>
      {/* ... */}
    </Chart>
  );
}
```

## Checking Import Cost

VS Code Extension: **Import Cost**

Shows bundle size impact inline:

```jsx
import moment from 'moment'; // 72.1KB (gzipped: 25.4KB)
import dayjs from 'dayjs';   // 2.8KB (gzipped: 1.2KB)
```

## Bundle Budget

```js
// vite.config.js
export default {
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          charts: ['recharts'],
        },
      },
    },
  },
};
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*