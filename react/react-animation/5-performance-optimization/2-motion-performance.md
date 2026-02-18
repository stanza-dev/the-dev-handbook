---
source_course: "react-animation"
source_lesson: "react-animation-motion-performance"
---

# Framer Motion Performance

Optimizing Framer Motion animations.

## useMotionValue for Direct Updates

```jsx
import { motion, useMotionValue, useTransform } from 'framer-motion';

// ❌ Re-renders on every update
function BadSlider() {
  const [x, setX] = useState(0);
  return (
    <motion.div
      drag="x"
      onDrag={(_, info) => setX(info.point.x)}
    />
  );
}

// ✅ No re-renders, direct DOM updates
function GoodSlider() {
  const x = useMotionValue(0);
  return (
    <motion.div
      drag="x"
      style={{ x }}
    />
  );
}
```

## useTransform for Derived Values

```jsx
function ParallaxCard() {
  const x = useMotionValue(0);
  
  // Derived values update without re-render
  const rotate = useTransform(x, [-200, 200], [-30, 30]);
  const opacity = useTransform(x, [-200, 0, 200], [0.5, 1, 0.5]);
  const background = useTransform(
    x,
    [-200, 0, 200],
    ['#ff0000', '#00ff00', '#0000ff']
  );

  return (
    <motion.div
      drag="x"
      style={{ x, rotate, opacity, background }}
      dragConstraints={{ left: -200, right: 200 }}
    />
  );
}
```

## useSpring for Smooth Values

```jsx
import { useSpring, useMotionValue } from 'framer-motion';

function SmoothFollower() {
  const x = useMotionValue(0);
  const smoothX = useSpring(x, {
    stiffness: 100,
    damping: 20,
  });

  return (
    <>
      <motion.div
        drag="x"
        style={{ x }}
        className="leader"
      />
      <motion.div
        style={{ x: smoothX }}
        className="follower"
      />
    </>
  );
}
```

## Reduce Motion Preference

```jsx
import { useReducedMotion } from 'framer-motion';

function AccessibleAnimation() {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      animate={{ x: 100 }}
      transition={{
        duration: shouldReduceMotion ? 0 : 0.5,
      }}
    />
  );
}

// Or apply globally
function App() {
  const shouldReduceMotion = useReducedMotion();
  
  return (
    <MotionConfig reducedMotion="user">
      {/* Your app */}
    </MotionConfig>
  );
}
```

## Lazy Loading Animations

```jsx
import { LazyMotion, domAnimation, m } from 'framer-motion';

// Only load animation features when needed
function App() {
  return (
    <LazyMotion features={domAnimation}>
      <m.div animate={{ opacity: 1 }} />
    </LazyMotion>
  );
}

// Even more minimal
import { LazyMotion, domMax } from 'framer-motion';

function App() {
  return (
    <LazyMotion features={domMax} strict>
      {/* Full features, loaded lazily */}
    </LazyMotion>
  );
}
```

## Avoid Re-renders During Animation

```jsx
// ❌ Bad: State changes cause re-renders
function BadAnimation() {
  const [scale, setScale] = useState(1);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setScale((s) => (s === 1 ? 1.2 : 1));
    }, 500);
    return () => clearInterval(interval);
  }, []);
  
  return <motion.div animate={{ scale }} />;
}

// ✅ Good: Use variants or motion values
function GoodAnimation() {
  return (
    <motion.div
      animate={{ scale: [1, 1.2, 1] }}
      transition={{ repeat: Infinity, duration: 1 }}
    />
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*