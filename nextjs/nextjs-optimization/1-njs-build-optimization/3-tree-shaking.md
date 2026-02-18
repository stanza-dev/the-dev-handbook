---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-tree-shaking"
---

# Tree Shaking and Dead Code Elimination

## Introduction

Tree shaking removes unused code from your bundles. Import the `format` function from a 100KB library, and only the code for `format` ships to users—not the entire library.

## Key Concepts

**Tree shaking** eliminates unused exports:
- Only imported functions are bundled
- Side-effect-free modules can be fully optimized
- ES modules enable better tree shaking than CommonJS

## Real World Context

Without tree shaking:
- Import one lodash function → ship 70KB
- Import one icon → ship entire icon library
- Every unused export still downloads

## Deep Dive

### Import Patterns Matter

```typescript
// Bad: Imports entire library
import _ from 'lodash';
_.get(obj, 'path');

// Good: Tree-shakeable import
import get from 'lodash/get';
get(obj, 'path');

// Better: Use native methods
obj?.path;
```

### Package.json sideEffects

Packages declare if they have side effects:

```json
{
  "sideEffects": false
}
```

If `false`, unused imports are completely removed.

### Analyzing What's Included

```bash
ANALYZE=true npm run build
```

Look for:
- Large dependencies you didn't expect
- Duplicate packages (different versions)
- Code that should be tree-shaken but isn't

### Common Library Optimizations

```typescript
// date-fns: Tree-shakeable
import { format } from 'date-fns';

// moment: NOT tree-shakeable (70KB+)
import moment from 'moment'; // Avoid!

// Icons: Import individually
import { IconHome } from '@tabler/icons-react';
// NOT: import * as Icons from '@tabler/icons-react';
```

## Common Pitfalls

1. **Barrel files**: `index.ts` re-exporting everything can break tree shaking in some bundlers.

2. **Dynamic imports breaking tree shaking**: `import(module)` can't be tree-shaken.

3. **Side effects in modules**: Code that runs on import prevents tree shaking.

## Best Practices

- **Check package size before installing**: Use bundlephobia.com
- **Import specifically**: `import { x } from 'lib'` not `import lib`
- **Use modern packages**: Prefer packages with ES module exports
- **Analyze regularly**: Run bundle analyzer with each major feature

## Summary

Tree shaking removes unused code from your bundles. Import functions directly rather than entire libraries, prefer ES module packages, and use the bundle analyzer to verify optimization. Small imports = small bundles = fast apps.

## Resources

- [Package Bundling](https://nextjs.org/docs/app/building-your-application/optimizing/package-bundling) — Next.js package bundling optimization

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*