---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-rtl-components"
---

# Building RTL-Aware Components

## Using CSS :dir() Selector

```css
.icon {
  transform: none;
}

.icon:dir(rtl) {
  transform: scaleX(-1); /* Mirror the icon */
}
```

## React Approach

```typescript
'use client';

import { useParams } from 'next/navigation';

const rtlLocales = ['ar', 'he', 'fa'];

export function useDirection() {
  const { lang } = useParams();
  return rtlLocales.includes(lang as string) ? 'rtl' : 'ltr';
}

export function Arrow() {
  const dir = useDirection();

  return (
    <span style={{ transform: dir === 'rtl' ? 'scaleX(-1)' : 'none' }}>
      →
    </span>
  );
}
```

## Tailwind RTL Plugin

```bash
npm install tailwindcss-rtl
```

```javascript
// tailwind.config.js
module.exports = {
  plugins: [require('tailwindcss-rtl')],
};
```

Then use `rtl:` and `ltr:` prefixes:

```html
<div class="ml-4 rtl:mr-4 rtl:ml-0">...</div>
```

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*