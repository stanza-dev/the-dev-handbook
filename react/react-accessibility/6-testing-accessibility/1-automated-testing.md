---
source_course: "react-accessibility"
source_lesson: "react-accessibility-automated-testing"
---

# Automated Accessibility Testing

Automate catching common accessibility issues.

## jest-axe

Test accessibility with Jest:

```bash
npm install --save-dev jest-axe
```

```jsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('form is accessible', async () => {
  const { container } = render(<LoginForm />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

## Testing Specific Rules

```jsx
test('images have alt text', async () => {
  const { container } = render(<Gallery />);
  
  const results = await axe(container, {
    rules: {
      'image-alt': { enabled: true }
    }
  });
  
  expect(results).toHaveNoViolations();
});
```

## ESLint Plugin

```bash
npm install --save-dev eslint-plugin-jsx-a11y
```

```js
// .eslintrc.js
module.exports = {
  plugins: ['jsx-a11y'],
  extends: ['plugin:jsx-a11y/recommended'],
};
```

Catches issues like:
- Missing alt text
- Invalid ARIA attributes
- Missing form labels
- Click handlers on non-interactive elements

## Storybook Accessibility Addon

```bash
npm install --save-dev @storybook/addon-a11y
```

```js
// .storybook/main.js
module.exports = {
  addons: ['@storybook/addon-a11y'],
};
```

Provides:
- Real-time accessibility panel
- Visual indicators for violations
- Colorblind simulation

## Lighthouse CI

```yaml
# .github/workflows/a11y.yml
name: Accessibility
on: [push]
jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Lighthouse
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            http://localhost:3000/
          temporaryPublicStorage: true
```

## What Automation Catches

✅ Can detect:
- Missing alt text
- Poor color contrast
- Missing form labels
- Invalid ARIA
- Missing language attribute
- Heading hierarchy issues

❌ Cannot detect:
- Meaningful alt text
- Logical tab order
- Keyboard usability
- Screen reader experience
- Focus management

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*