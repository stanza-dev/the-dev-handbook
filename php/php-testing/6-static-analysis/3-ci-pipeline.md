---
source_course: "php-testing"
source_lesson: "php-testing-ci-pipeline"
---

# CI/CD Pipeline for Testing

## Introduction
A CI/CD pipeline runs your tests, static analysis, and code style checks automatically on every push and pull request. This ensures that no broken code reaches your main branch. GitHub Actions is the most popular CI platform for PHP projects, and setting it up takes just a single YAML file.

## Key Concepts
- **CI (Continuous Integration)**: The practice of automatically building and testing code every time changes are pushed to a shared repository.
- **GitHub Actions**: GitHub's built-in CI/CD platform that runs workflows defined in YAML files inside `.github/workflows/`.
- **Matrix Strategy**: Running the same test suite across multiple PHP versions or operating systems in parallel.
- **Pipeline**: A sequence of automated steps (install, lint, test, analyze) that code must pass before it can be merged.

## Real World Context
Without CI, developers must manually run tests before merging, and someone always forgets. CI eliminates this risk by making tests a gate: pull requests cannot be merged until all checks pass. Most open-source PHP libraries use GitHub Actions, and it is free for public repositories.

## Deep Dive

### Basic GitHub Actions Workflow
Create a workflow file at `.github/workflows/ci.yml`. This runs PHPUnit, PHPStan, and PHP CS Fixer on every push.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.5'
          extensions: mbstring, pdo_mysql
          coverage: xdebug

      - name: Install dependencies
        run: composer install --prefer-dist --no-progress

      - name: Run PHPStan
        run: ./vendor/bin/phpstan analyse

      - name: Check code style
        run: ./vendor/bin/php-cs-fixer fix --dry-run --diff

      - name: Run tests
        run: ./vendor/bin/phpunit --coverage-text
```

This workflow has a single job called `test` with five steps: checkout the code, set up PHP 8.5, install Composer dependencies, run static analysis, check code style, and run tests with coverage output.

### Matrix Testing Across PHP Versions
To ensure your code works on multiple PHP versions, use a matrix strategy.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        php-version: ['8.3', '8.4', '8.5']

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP ${{ matrix.php-version }}
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-version }}
          extensions: mbstring, pdo_mysql

      - name: Install dependencies
        run: composer install --prefer-dist --no-progress

      - name: Run tests
        run: ./vendor/bin/phpunit
```

This creates three parallel jobs, one for each PHP version. If tests fail on PHP 8.3 but pass on 8.5, you will know immediately which version has the compatibility issue.

### Separating Quality Checks
For larger projects, split quality checks into separate jobs so they run in parallel and you can see exactly which step failed.

```yaml
jobs:
  phpstan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.5'
      - run: composer install --prefer-dist --no-progress
      - run: ./vendor/bin/phpstan analyse

  code-style:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.5'
      - run: composer install --prefer-dist --no-progress
      - run: ./vendor/bin/php-cs-fixer fix --dry-run --diff

  tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        php-version: ['8.3', '8.4', '8.5']
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-version }}
      - run: composer install --prefer-dist --no-progress
      - run: ./vendor/bin/phpunit
```

With separate jobs, PHPStan, PHP CS Fixer, and PHPUnit all run simultaneously. If code style fails, you do not have to wait for tests to finish to see the failure.

### Caching Dependencies
Composer installation can be slow. Cache the vendor directory to speed up builds.

```yaml
- name: Cache Composer dependencies
  uses: actions/cache@v4
  with:
    path: vendor
    key: ${{ runner.os }}-composer-${{ hashFiles('composer.lock') }}
    restore-keys: ${{ runner.os }}-composer-

- name: Install dependencies
  run: composer install --prefer-dist --no-progress
```

This caches the `vendor` directory based on the `composer.lock` hash. If dependencies have not changed, the install step takes seconds instead of minutes.

## Common Pitfalls
1. **Not caching Composer dependencies** — Without caching, every CI run downloads all packages from scratch. This wastes time and can hit rate limits. Always cache the vendor directory.
2. **Running only tests, not static analysis** — Tests verify behavior but miss type errors and code smells. A complete CI pipeline includes PHPUnit, PHPStan, and PHP CS Fixer to catch different categories of issues.

## Best Practices
1. **Make CI a required status check** — In GitHub repository settings, mark the CI workflow as required for pull request merges. This prevents anyone from merging code that fails tests or analysis.
2. **Test against the minimum supported PHP version** — Always include your lowest supported PHP version in the matrix. Bugs often appear on older versions due to missing features or different behavior.

## Summary
- GitHub Actions runs tests and quality checks automatically on every push and pull request.
- A basic CI pipeline includes PHPUnit, PHPStan, and PHP CS Fixer.
- Matrix testing ensures compatibility across multiple PHP versions.
- Separating quality checks into parallel jobs provides faster feedback and clearer error reporting.
- Caching Composer dependencies dramatically speeds up CI builds.

## Code Examples

**Complete GitHub Actions workflow for PHP quality assurance: matrix testing across PHP 8.3-8.5 with dependency caching, PHPStan, CS Fixer, and PHPUnit**

```yaml
# .github/workflows/ci.yml
name: PHP Quality Assurance

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        php-version: ['8.3', '8.4', '8.5']

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP ${{ matrix.php-version }}
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-version }}
          extensions: mbstring, pdo_mysql
          coverage: xdebug

      - name: Cache Composer dependencies
        uses: actions/cache@v4
        with:
          path: vendor
          key: ${{ runner.os }}-composer-${{ hashFiles('composer.lock') }}

      - name: Install dependencies
        run: composer install --prefer-dist --no-progress

      - name: Run PHPStan
        run: ./vendor/bin/phpstan analyse

      - name: Check code style
        run: ./vendor/bin/php-cs-fixer fix --dry-run --diff

      - name: Run tests with coverage
        run: ./vendor/bin/phpunit --coverage-text
```


## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions) — Official GitHub Actions documentation for setting up CI/CD workflows
- [Setup PHP Action](https://github.com/shivammathur/setup-php) — The most popular GitHub Action for setting up PHP with extensions and tools

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*