---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-nx-basics"
---

# Nx for Monorepos

Nx provides powerful monorepo tooling.

## Setup

```bash
# Create new Nx workspace
npx create-nx-workspace@latest my-workspace

# Or add to existing repo
npx nx@latest init
```

## Project Structure

```
my-workspace/
├── apps/
│   ├── web/
│   │   ├── src/
│   │   └── project.json
│   └── web-e2e/
├── libs/
│   ├── ui/
│   │   ├── src/
│   │   └── project.json
│   └── utils/
├── nx.json
├── package.json
└── tsconfig.base.json
```

## Running Tasks

```bash
# Run single project task
nx build web
nx test ui
nx serve web

# Run task for all projects
nx run-many -t build
nx run-many -t test

# Run affected (changed) projects only
nx affected -t test
nx affected -t build
```

## Task Caching

Nx caches task results:

```bash
# First run: executes
nx build ui
# [4.2s]

# Second run: cached
nx build ui
# [0.1s] (cached)
```

## project.json

```json
// libs/ui/project.json
{
  "name": "ui",
  "$schema": "../../node_modules/nx/schemas/project-schema.json",
  "sourceRoot": "libs/ui/src",
  "projectType": "library",
  "targets": {
    "build": {
      "executor": "@nx/vite:build",
      "outputs": ["{options.outputPath}"],
      "options": {
        "outputPath": "dist/libs/ui"
      }
    },
    "test": {
      "executor": "@nx/jest:jest",
      "options": {
        "jestConfig": "libs/ui/jest.config.ts"
      }
    }
  },
  "tags": ["scope:shared", "type:ui"]
}
```

## Generators

```bash
# Generate new library
nx g @nx/react:library ui-components --directory=libs/ui-components

# Generate component
nx g @nx/react:component Button --project=ui-components

# Generate app
nx g @nx/next:application admin --directory=apps/admin
```

## Dependency Graph

```bash
# Visualize dependencies
nx graph
```

## Module Boundaries

```json
// nx.json
{
  "targetDefaults": {
    "build": {
      "cache": true
    }
  },
  "namedInputs": {
    "default": ["{projectRoot}/**/*"]
  }
}
```

```js
// .eslintrc.js
module.exports = {
  rules: {
    '@nx/enforce-module-boundaries': [
      'error',
      {
        allow: [],
        depConstraints: [
          {
            sourceTag: 'scope:web',
            onlyDependOnLibsWithTags: ['scope:shared'],
          },
          {
            sourceTag: 'type:feature',
            onlyDependOnLibsWithTags: ['type:ui', 'type:util'],
          },
        ],
      },
    ],
  },
};
```

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*