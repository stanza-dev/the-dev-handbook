---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-rtl-icons"
---

# RTL Icons and Images

## Introduction

Directional icons (arrows, chevrons) need to flip in RTL. But not all icons should flip—a clock still rotates clockwise everywhere.

## Key Concepts

**Icon categories**:

- Directional: Arrows, chevrons, reply icons → flip
- Non-directional: Search, settings, clock → don't flip
- Text-related: Quote marks, bullet points → may need flip

## Deep Dive

### CSS Flipping

```css
/* Flip icon in RTL */
.icon-arrow:dir(rtl) {
  transform: scaleX(-1);
}

/* Using Tailwind RTL plugin */
<ChevronRight className="rtl:-scale-x-100" />
```

### React Component

```typescript
function DirectionalIcon({ 
  icon: Icon, 
  className 
}: { 
  icon: React.ComponentType<{ className?: string }>;
  className?: string;
}) {
  const dir = useDirection();
  
  return (
    <Icon 
      className={cn(
        className,
        dir === 'rtl' && 'scale-x-[-1]'
      )} 
    />
  );
}

// Usage
<DirectionalIcon icon={ArrowRight} />
```

### Icons That Should NOT Flip

```typescript
const NON_DIRECTIONAL_ICONS = [
  'search',
  'settings',
  'clock',
  'calendar',
  'phone',
  'play',  // Media control
  'pause',
];
```

### Image Considerations

```tsx
// Screenshots may need locale-specific versions
<Image
  src={`/screenshots/${locale}/dashboard.png`}
  alt={t('dashboard.screenshot')}
/>
```

## Summary

Flip directional icons like arrows in RTL using CSS transforms. Keep non-directional icons unchanged. Consider providing locale-specific screenshots for marketing pages.

## Resources

- [RTL Styling](https://rtlstyling.com/) — Comprehensive RTL styling guide

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*