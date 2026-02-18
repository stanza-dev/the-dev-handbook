---
source_course: "react-animation"
source_lesson: "react-animation-css-transitions"
---

# CSS Transitions in React

CSS transitions are the simplest way to animate in React.

## Basic Transitions

Transitions animate property changes smoothly:

```jsx
function Button() {
  const [isHovered, setIsHovered] = useState(false);

  return (
    <button
      className={`btn ${isHovered ? 'hovered' : ''}`}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      Hover me
    </button>
  );
}
```

```css
.btn {
  background: #3b82f6;
  transform: scale(1);
  transition: transform 200ms ease, background 200ms ease;
}

.btn.hovered {
  background: #2563eb;
  transform: scale(1.05);
}
```

## Transition Properties

```css
/* Individual properties */
.element {
  transition-property: opacity, transform;
  transition-duration: 300ms;
  transition-timing-function: ease-out;
  transition-delay: 0ms;
}

/* Shorthand */
.element {
  transition: opacity 300ms ease-out, transform 300ms ease-out;
}

/* All properties (use sparingly) */
.element {
  transition: all 300ms ease-out;
}
```

## Timing Functions

```css
.element {
  /* Built-in functions */
  transition-timing-function: ease;        /* Default, slow-fast-slow */
  transition-timing-function: ease-in;     /* Starts slow */
  transition-timing-function: ease-out;    /* Ends slow */
  transition-timing-function: ease-in-out; /* Slow start and end */
  transition-timing-function: linear;      /* Constant speed */
  
  /* Custom cubic-bezier */
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}
```

## State-Driven Animations

```jsx
function Card({ isExpanded }) {
  return (
    <div
      style={{
        height: isExpanded ? '300px' : '100px',
        opacity: isExpanded ? 1 : 0.8,
        transition: 'height 300ms ease, opacity 300ms ease'
      }}
    >
      Card content
    </div>
  );
}
```

## With Tailwind CSS

```jsx
function AnimatedCard({ isActive }) {
  return (
    <div
      className={`
        transition-all duration-300 ease-out
        ${isActive 
          ? 'scale-105 shadow-xl bg-blue-500' 
          : 'scale-100 shadow-md bg-white'
        }
      `}
    >
      Content
    </div>
  );
}
```

## Performance Tips

```css
/* ✅ GPU-accelerated (fast) */
.element {
  transition: transform 300ms, opacity 300ms;
}

/* ❌ Triggers layout (slow) */
.element {
  transition: width 300ms, height 300ms, top 300ms;
}
```

Animate these for best performance:
- `transform` (translate, scale, rotate)
- `opacity`

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*