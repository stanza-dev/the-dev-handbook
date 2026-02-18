---
source_course: "react-animation"
source_lesson: "react-animation-reduced-motion"
---

# Respecting Reduced Motion

Some users need reduced or no motion for medical reasons.

## Why It Matters

- Vestibular disorders affect ~35% of adults over 40
- Motion can cause dizziness, nausea, headaches
- Some users have motion sensitivity
- Required for WCAG 2.1 Level AA compliance

## prefers-reduced-motion Media Query

```css
/* Default animation */
.animated {
  animation: slide-in 300ms ease;
}

/* Reduce or remove for those who prefer */
@media (prefers-reduced-motion: reduce) {
  .animated {
    animation: none;
    /* Or use a simpler animation */
    animation: fade-in 150ms ease;
  }
}
```

## React Hook for Reduced Motion

```jsx
function useReducedMotion() {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia(
      '(prefers-reduced-motion: reduce)'
    );
    
    setPrefersReducedMotion(mediaQuery.matches);
    
    const handler = (e) => setPrefersReducedMotion(e.matches);
    mediaQuery.addEventListener('change', handler);
    
    return () => mediaQuery.removeEventListener('change', handler);
  }, []);

  return prefersReducedMotion;
}

// Usage
function AnimatedComponent() {
  const prefersReducedMotion = useReducedMotion();

  return (
    <div
      style={{
        transition: prefersReducedMotion
          ? 'none'
          : 'transform 300ms ease',
      }}
    >
      Content
    </div>
  );
}
```

## Framer Motion's useReducedMotion

```jsx
import { motion, useReducedMotion } from 'framer-motion';

function Card() {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      initial={{ opacity: 0, y: shouldReduceMotion ? 0 : 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{
        duration: shouldReduceMotion ? 0.1 : 0.3,
      }}
    >
      Card content
    </motion.div>
  );
}
```

## MotionConfig for Global Settings

```jsx
import { MotionConfig } from 'framer-motion';

function App() {
  return (
    <MotionConfig reducedMotion="user">
      {/* All motion components respect user preference */}
      <Main />
    </MotionConfig>
  );
}

// Options:
// "user" - respect system preference (default)
// "always" - always reduce motion
// "never" - ignore preference
```

## Safe vs Problematic Animations

```jsx
// ✅ Safe animations (usually okay to keep)
// - Opacity fades
// - Color changes
// - Small scale changes
// - Border/shadow changes

// ⚠️ Potentially problematic (reduce or remove)
// - Large translations
// - Rotations
// - Parallax scrolling
// - Flashing/blinking
// - Zooming
// - Complex sequences
```

## Implementation Pattern

```jsx
const animationVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 },
};

const reducedMotionVariants = {
  hidden: { opacity: 0 },
  visible: { opacity: 1 },
};

function AdaptiveAnimation({ children }) {
  const shouldReduceMotion = useReducedMotion();
  
  const variants = shouldReduceMotion
    ? reducedMotionVariants
    : animationVariants;

  return (
    <motion.div
      variants={variants}
      initial="hidden"
      animate="visible"
      transition={{ duration: shouldReduceMotion ? 0.1 : 0.3 }}
    >
      {children}
    </motion.div>
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*