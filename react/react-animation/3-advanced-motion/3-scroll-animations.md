---
source_course: "react-animation"
source_lesson: "react-animation-scroll-animations"
---

# Scroll-Triggered Animations

Animate elements based on scroll position.

## whileInView

```jsx
function ScrollReveal({ children }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 50 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true, margin: '-100px' }}
      transition={{ duration: 0.6 }}
    >
      {children}
    </motion.div>
  );
}
```

## Viewport Options

```jsx
<motion.div
  whileInView={{ opacity: 1 }}
  viewport={{
    once: true,           // Only animate once
    margin: '-100px',     // Trigger margin
    amount: 0.5,          // 0-1, portion visible to trigger
    root: scrollRef,      // Custom scroll container
  }}
/>
```

## Scroll Progress Animation

```jsx
import { motion, useScroll, useTransform } from 'framer-motion';

function ProgressBar() {
  const { scrollYProgress } = useScroll();

  return (
    <motion.div
      className="progress-bar"
      style={{ scaleX: scrollYProgress }}
    />
  );
}
```

## Element Scroll Progress

```jsx
function ParallaxImage({ src }) {
  const ref = useRef(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start end', 'end start'],
  });
  
  const y = useTransform(scrollYProgress, [0, 1], ['-20%', '20%']);

  return (
    <div ref={ref} className="parallax-container">
      <motion.img
        src={src}
        style={{ y }}
        className="parallax-image"
      />
    </div>
  );
}
```

## Parallax Section

```jsx
function ParallaxHero() {
  const { scrollY } = useScroll();
  const y = useTransform(scrollY, [0, 500], [0, 150]);
  const opacity = useTransform(scrollY, [0, 300], [1, 0]);

  return (
    <motion.section style={{ y, opacity }} className="hero">
      <h1>Welcome</h1>
      <p>Scroll down to explore</p>
    </motion.section>
  );
}
```

## Scroll-Linked Animations

```jsx
import { useScroll, useSpring, useTransform } from 'framer-motion';

function ScrollLinkedCard() {
  const ref = useRef(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start end', 'center center'],
  });

  const scale = useTransform(scrollYProgress, [0, 1], [0.8, 1]);
  const rotateX = useTransform(scrollYProgress, [0, 1], [20, 0]);
  
  // Add spring for smooth movement
  const smoothScale = useSpring(scale, { stiffness: 100, damping: 20 });

  return (
    <motion.div
      ref={ref}
      style={{
        scale: smoothScale,
        rotateX,
        transformPerspective: 1000,
      }}
      className="card"
    >
      Content
    </motion.div>
  );
}
```

## Staggered Scroll Reveal

```jsx
function StaggeredScrollList({ items }) {
  return (
    <motion.ul
      initial="hidden"
      whileInView="visible"
      viewport={{ once: true }}
      variants={{
        visible: {
          transition: { staggerChildren: 0.1 },
        },
      }}
    >
      {items.map((item) => (
        <motion.li
          key={item.id}
          variants={{
            hidden: { opacity: 0, x: -20 },
            visible: { opacity: 1, x: 0 },
          }}
        >
          {item.name}
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*