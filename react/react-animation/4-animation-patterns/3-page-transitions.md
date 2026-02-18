---
source_course: "react-animation"
source_lesson: "react-animation-page-transitions"
---

# Page Transitions

Smooth transitions between routes enhance navigation.

## Basic Page Transition

```jsx
import { motion } from 'framer-motion';

const pageVariants = {
  initial: { opacity: 0, x: -20 },
  animate: { opacity: 1, x: 0 },
  exit: { opacity: 0, x: 20 },
};

function PageWrapper({ children }) {
  return (
    <motion.div
      variants={pageVariants}
      initial="initial"
      animate="animate"
      exit="exit"
      transition={{ duration: 0.3 }}
    >
      {children}
    </motion.div>
  );
}

// Usage in each page
function HomePage() {
  return (
    <PageWrapper>
      <h1>Home</h1>
    </PageWrapper>
  );
}
```

## App Router Setup

```jsx
import { AnimatePresence } from 'framer-motion';
import { useLocation, Routes, Route } from 'react-router-dom';

function App() {
  const location = useLocation();

  return (
    <AnimatePresence mode="wait">
      <Routes location={location} key={location.pathname}>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="/contact" element={<ContactPage />} />
      </Routes>
    </AnimatePresence>
  );
}
```

## Slide Transitions

```jsx
const slideVariants = {
  enter: (direction) => ({
    x: direction > 0 ? '100%' : '-100%',
    opacity: 0,
  }),
  center: {
    x: 0,
    opacity: 1,
  },
  exit: (direction) => ({
    x: direction < 0 ? '100%' : '-100%',
    opacity: 0,
  }),
};

function SlideTransition({ direction, children }) {
  return (
    <motion.div
      custom={direction}
      variants={slideVariants}
      initial="enter"
      animate="center"
      exit="exit"
      transition={{ type: 'tween', duration: 0.3 }}
    >
      {children}
    </motion.div>
  );
}
```

## Fade Through Transition

```jsx
const fadeThrough = {
  initial: { opacity: 0, scale: 0.96 },
  animate: {
    opacity: 1,
    scale: 1,
    transition: {
      duration: 0.3,
      ease: [0.4, 0, 0.2, 1],
    },
  },
  exit: {
    opacity: 0,
    scale: 1.04,
    transition: {
      duration: 0.2,
      ease: [0.4, 0, 1, 1],
    },
  },
};
```

## Shared Axis Transition

```jsx
const sharedAxis = {
  initial: { opacity: 0, y: 30 },
  animate: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.4, ease: [0, 0, 0.2, 1] },
  },
  exit: {
    opacity: 0,
    y: -30,
    transition: { duration: 0.3, ease: [0.4, 0, 1, 1] },
  },
};
```

## Combined with Navigation

```jsx
import { useNavigate, useLocation } from 'react-router-dom';

function useNavigateWithTransition() {
  const navigate = useNavigate();
  const [direction, setDirection] = useState(0);

  const navigateTo = (path, newDirection = 1) => {
    setDirection(newDirection);
    navigate(path);
  };

  return { navigateTo, direction };
}

function Navigation() {
  const { navigateTo } = useNavigateWithTransition();

  return (
    <nav>
      <button onClick={() => navigateTo('/', -1)}>Home</button>
      <button onClick={() => navigateTo('/about', 1)}>About</button>
    </nav>
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*