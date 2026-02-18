---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-bidirectional-text"
---

# Bidirectional Text

## Introduction

Arabic text with English words, or Hebrew with numbers, creates "bidirectional" or "bidi" text. Browsers handle most cases, but you need to help with ambiguous situations.

## Key Concepts

**Bidi challenges**:

- Numbers in RTL text
- Mixed language content
- Punctuation placement
- User-generated content

## Deep Dive

### The dir Attribute

```html
<!-- Force RTL for Arabic content -->
<p dir="rtl">مرحبا بالعالم</p>

<!-- Auto-detect direction -->
<p dir="auto">{userContent}</p>
```

### Isolating Text

```html
<!-- Isolate LTR text in RTL context -->
<p dir="rtl">
  السعر: <bdi>$100 USD</bdi>
</p>

<!-- Or with CSS -->
<span style="unicode-bidi: isolate">$100 USD</span>
```

### React Component

```typescript
function BidiText({ 
  children,
  dir = 'auto'
}: { 
  children: React.ReactNode;
  dir?: 'ltr' | 'rtl' | 'auto';
}) {
  return <span dir={dir}>{children}</span>;
}

// Usage for user content
<BidiText dir="auto">{comment.text}</BidiText>
```

### Common Issues

```typescript
// Problem: Punctuation in wrong place
// "Hello, world!" → "!Hello, world" in RTL context

// Solution: Use bdi or isolate
<bdi>Hello, world!</bdi>
```

## Summary

Use `dir="auto"` for user-generated content and `<bdi>` or `unicode-bidi: isolate` to isolate LTR content within RTL text. Test with real mixed-language content.

## Resources

- [Unicode Bidirectional Algorithm](https://www.w3.org/International/articles/inline-bidi-markup/) — W3C guide to bidirectional markup

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*