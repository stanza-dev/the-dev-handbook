---
source_course: "nextjs"
source_lesson: "nextjs-next-image-component"
---

# Image Optimization

The Next.js Image component (`<Image>`) extends the HTML `<img>` element with automatic optimization features.

## Features

*   **Size Optimization**: Automatically serves correctly sized images for each device.
*   **Visual Stability**: Prevents Layout Shift automatically.
*   **Faster Page Loads**: Images are only loaded when they enter the viewport (lazy loading).

## Usage

```tsx
import Image from 'next/image'
import profilePic from './profile.png'

export default function Page() {
  return (
    <Image
      src={profilePic}
      alt="Picture of the author"
      // width/height optional for local imports
    />
  )
}
```

For remote images (strings), you must provide `width` and `height` props.

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*