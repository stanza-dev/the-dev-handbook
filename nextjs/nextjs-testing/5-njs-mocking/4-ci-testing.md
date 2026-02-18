---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-ci-testing"
---

# Testing in CI/CD

## Introduction

Tests that pass locally but fail in CI are frustrating. Setting up reliable CI testing ensures consistent results across environments.

## Key Concepts

**CI testing best practices**:

- Consistent environments (Docker, Node versions)
- Parallel test execution
- Test result caching
- Fail fast strategies

## Deep Dive

### GitHub Actions Workflow

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Run unit tests
        run: pnpm test
      
      - name: Run E2E tests
        run: pnpm exec playwright test
```

### Playwright in CI

```yaml
- name: Install Playwright Browsers
  run: pnpm exec playwright install --with-deps

- name: Run Playwright tests
  run: pnpm exec playwright test

- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: playwright-report
    path: playwright-report/
```

### Caching Dependencies

```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.pnpm-store
      node_modules
    key: ${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}
```

## Summary

Reliable CI testing requires consistent environments, proper caching, and artifact storage for debugging failures. Upload test reports and screenshots as artifacts for easier debugging.

## Resources

- [GitHub Actions](https://docs.github.com/en/actions) — GitHub Actions documentation

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*