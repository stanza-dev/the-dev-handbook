---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-flat-vs-nested"
---

# Flat vs Nested Structures

Choose based on project size and team needs.

## Flat Structure (Small Projects)

```
src/
├── components/
│   ├── Button.tsx
│   ├── Card.tsx
│   ├── Header.tsx
│   ├── Modal.tsx
│   └── Sidebar.tsx
├── hooks/
│   ├── useAuth.ts
│   └── useFetch.ts
├── utils/
│   └── helpers.ts
├── App.tsx
└── index.tsx
```

**Pros**: Simple, easy to find files
**Cons**: Doesn't scale, hard to find related code

## Nested by Type (Medium Projects)

```
src/
├── components/
│   ├── common/
│   │   ├── Button/
│   │   └── Modal/
│   └── layout/
│       ├── Header/
│       └── Sidebar/
├── pages/
│   ├── Home/
│   ├── Dashboard/
│   └── Settings/
├── hooks/
├── services/
├── types/
└── utils/
```

**Pros**: Organized by type
**Cons**: Related code is scattered

## Feature-Based (Large Projects)

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── types.ts
│   │   └── index.ts
│   ├── dashboard/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── index.ts
│   └── settings/
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
├── app/
│   ├── routes.tsx
│   ├── store.ts
│   └── App.tsx
└── index.tsx
```

**Pros**: Related code together, easy to find
**Cons**: More structure to maintain

## Choosing a Structure

| Project Size | Team Size | Recommendation |
|-------------|-----------|----------------|
| < 10 components | 1-2 devs | Flat |
| 10-50 components | 2-5 devs | Nested by type |
| 50+ components | 5+ devs | Feature-based |

## Component Folder Structure

```
Button/
├── Button.tsx       # Main component
├── Button.test.tsx  # Tests
├── Button.stories.tsx # Storybook
├── Button.module.css  # Styles
├── types.ts         # Types (if complex)
└── index.ts         # Re-exports
```

```tsx
// Button/index.ts
export { Button } from './Button';
export type { ButtonProps } from './types';
```

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*