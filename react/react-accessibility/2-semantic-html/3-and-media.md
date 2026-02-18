---
source_course: "react-accessibility"
source_lesson: "react-accessibility-images-and-media"
---

# Images & Media Accessibility

All non-text content needs text alternatives.

## Alt Text for Images

```jsx
// Informative image - describe content
<img 
  src="chart.png" 
  alt="Bar chart showing sales increased 50% from Q3 to Q4"
/>

// Decorative image - empty alt
<img src="decorative-border.png" alt="" />

// Image as link - describe destination
<a href="/profile">
  <img src="avatar.jpg" alt="View your profile" />
</a>

// Complex image - longer description
<figure>
  <img 
    src="infographic.png" 
    alt="Company growth infographic"
    aria-describedby="infographic-desc"
  />
  <figcaption id="infographic-desc">
    Detailed description of the infographic data...
  </figcaption>
</figure>
```

## Writing Good Alt Text

```jsx
// ❌ Bad: Filename or generic
<img src="IMG_1234.jpg" alt="image" />
<img src="photo.jpg" alt="photo of person" />

// ❌ Bad: Redundant
<img src="logo.png" alt="Image of our company logo" />

// ✅ Good: Descriptive, concise
<img src="logo.png" alt="Acme Corporation" />
<img src="team.jpg" alt="Our team at the 2024 conference" />
```

## Icons

```jsx
// Icon-only button - needs accessible name
<button aria-label="Close dialog">
  <CloseIcon aria-hidden="true" />
</button>

// Icon with visible text - hide icon from AT
<button>
  <SaveIcon aria-hidden="true" />
  <span>Save</span>
</button>

// Informative icon - include in text
<span>
  <WarningIcon aria-hidden="true" />
  <span>Warning: This action cannot be undone</span>
</span>
```

## SVG Accessibility

```jsx
// Decorative SVG
<svg aria-hidden="true" focusable="false">
  {/* paths */}
</svg>

// Informative SVG
<svg role="img" aria-labelledby="chart-title">
  <title id="chart-title">Monthly Revenue Chart</title>
  {/* paths */}
</svg>
```

## Video & Audio

```jsx
<video controls>
  <source src="video.mp4" type="video/mp4" />
  {/* Captions */}
  <track
    kind="captions"
    src="captions.vtt"
    srcLang="en"
    label="English"
    default
  />
  {/* Audio description for blind users */}
  <track
    kind="descriptions"
    src="descriptions.vtt"
    srcLang="en"
    label="Audio Descriptions"
  />
</video>

{/* Transcript for all users */}
<details>
  <summary>Video Transcript</summary>
  <p>Full transcript text...</p>
</details>
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*