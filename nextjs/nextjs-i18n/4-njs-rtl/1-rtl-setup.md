---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-rtl-setup"
---

# RTL Support

Languages like Arabic, Hebrew, and Persian are written right-to-left (RTL) and require special handling.

## Setting the `dir` Attribute

```typescript
// app/[lang]/layout.tsx
const rtlLocales = ['ar', 'he', 'fa'];

export default function Layout({
  children,
  params: { lang },
}: {
  children: React.ReactNode;
  params: { lang: string };
}) {
  const dir = rtlLocales.includes(lang) ? 'rtl' : 'ltr';

  return (
    <html lang={lang} dir={dir}>
      <body>{children}</body>
    </html>
  );
}
```

## CSS Considerations

Use logical properties instead of physical ones:

```css
/* Physical (avoid) */
.box {
  margin-left: 20px;
  padding-right: 10px;
  text-align: left;
}

/* Logical (recommended) */
.box {
  margin-inline-start: 20px;
  padding-inline-end: 10px;
  text-align: start;
}
```

## Logical Property Mapping

| Physical | Logical |
|----------|----------|
| left | inline-start |
| right | inline-end |
| top | block-start |
| bottom | block-end |
| margin-left | margin-inline-start |
| padding-right | padding-inline-end |

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*