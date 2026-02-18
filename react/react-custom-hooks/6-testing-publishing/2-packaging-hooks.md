---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-packaging-hooks"
---

# Packaging and Sharing Hooks

Share your hooks as an npm package.

## Project Structure

```
my-hooks/
├── src/
│   ├── useToggle.ts
│   ├── useLocalStorage.ts
│   ├── useMediaQuery.ts
│   └── index.ts
├── tests/
│   ├── useToggle.test.ts
│   └── ...
├── package.json
├── tsconfig.json
└── README.md
```

## Package.json

```json
{
  "name": "@yourname/react-hooks",
  "version": "1.0.0",
  "description": "Useful React hooks",
  "main": "dist/index.js",
  "module": "dist/index.esm.js",
  "types": "dist/index.d.ts",
  "files": ["dist"],
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "test": "jest",
    "prepublishOnly": "npm run build"
  },
  "peerDependencies": {
    "react": ">=17.0.0"
  },
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@types/react": "^18.0.0",
    "react": "^18.0.0",
    "tsup": "^7.0.0",
    "typescript": "^5.0.0"
  },
  "keywords": ["react", "hooks"],
  "license": "MIT"
}
```

## Entry Point (index.ts)

```tsx
// Export all hooks
export { useToggle } from './useToggle';
export { useLocalStorage } from './useLocalStorage';
export { useMediaQuery } from './useMediaQuery';
export { useDebounce } from './useDebounce';

// Export types
export type { UseToggleReturn } from './useToggle';
export type { UseLocalStorageOptions } from './useLocalStorage';
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2018",
    "module": "ESNext",
    "lib": ["DOM", "ES2018"],
    "declaration": true,
    "strict": true,
    "moduleResolution": "node",
    "esModuleInterop": true,
    "jsx": "react-jsx",
    "outDir": "dist"
  },
  "include": ["src"]
}
```

## Documentation

````markdown
# @yourname/react-hooks

Useful React hooks.

## Installation

```bash
npm install @yourname/react-hooks
```

## Hooks

### useToggle

```jsx
import { useToggle } from '@yourname/react-hooks';

function Component() {
  const [isOn, toggle] = useToggle(false);
  return <button onClick={toggle}>{isOn ? 'ON' : 'OFF'}</button>;
}
```

### useLocalStorage

```jsx
import { useLocalStorage } from '@yourname/react-hooks';

function Component() {
  const [name, setName] = useLocalStorage('name', '');
  return <input value={name} onChange={e => setName(e.target.value)} />;
}
```
````

## Publishing

```bash
# Login to npm
npm login

# Publish
npm publish --access public

# Or for scoped package
npm publish
```

## Version Management

```bash
# Bump version
npm version patch  # 1.0.0 -> 1.0.1
npm version minor  # 1.0.1 -> 1.1.0
npm version major  # 1.1.0 -> 2.0.0

# Then publish
npm publish
```

## Changelog

Keep a CHANGELOG.md:

```markdown
# Changelog

## [1.1.0] - 2024-01-15
### Added
- useMediaQuery hook

### Fixed
- useLocalStorage SSR support

## [1.0.0] - 2024-01-01
- Initial release
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*