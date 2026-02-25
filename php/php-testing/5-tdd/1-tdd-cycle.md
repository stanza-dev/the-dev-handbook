---
source_course: "php-testing"
source_lesson: "php-testing-tdd-cycle"
---

# The TDD Cycle: Red-Green-Refactor

## Introduction
Test-Driven Development (TDD) flips traditional development on its head: you write the test before the code it tests. This discipline forces you to think about what your code should do before deciding how it does it, leading to cleaner designs and fewer bugs.

## Key Concepts
- **TDD (Test-Driven Development)**: A development practice where you write a failing test first, then write just enough production code to make it pass, then refactor.
- **Red Phase**: Write a test that fails because the feature does not exist yet.
- **Green Phase**: Write the minimal production code necessary to make the failing test pass.
- **Refactor Phase**: Improve the code's structure and clarity without changing its behavior, keeping all tests green.
- **Red-Green-Refactor**: The three-step cycle that is the heartbeat of TDD.

## Real World Context
TDD is used at companies like Pivotal Labs, ThoughtWorks, and across the Symfony ecosystem. When a team practices TDD, every line of production code has a corresponding test, which dramatically reduces regression bugs. If you have ever spent hours debugging a change that broke something unrelated, TDD is the antidote. It also serves as living documentation: your tests describe exactly what the system is supposed to do.

## Deep Dive
The TDD cycle has three distinct phases that repeat continuously as you build features.

### Phase 1: Red (Write a Failing Test)
You begin by writing a test for behavior that does not exist yet. This test must fail. If it passes immediately, something is wrong with the test.

```php
<?php
use PHPUnit\Framework\TestCase;

class EmailValidatorTest extends TestCase
{
    public function testValidEmailReturnsTrue(): void
    {
        $validator = new EmailValidator();

        // This test will fail because EmailValidator does not exist yet
        $this->assertTrue($validator->isValid('user@example.com'));
    }
}
```

Running this test produces a fatal error: `Class 'EmailValidator' not found`. That is exactly the Red phase. The failure confirms your test is actually testing something real.

### Phase 2: Green (Make It Pass)
Now write the simplest code that makes the test pass. Do not worry about elegance.

```php
<?php
class EmailValidator
{
    public function isValid(string $email): bool
    {
        return true; // Simplest thing that passes
    }
}
```

This may seem absurd, but it is intentional. The Green phase is about satisfying the test, nothing more. You will add more tests to force a real implementation.

### Phase 3: Refactor (Clean Up)
With the test passing, you can safely restructure the code. You might extract methods, rename variables, or remove duplication. The key rule: the tests must stay green.

```php
<?php
class EmailValidator
{
    public function isValid(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
}
```

Now the implementation is real. You refactored the stub into proper validation logic, and the test still passes.

### When TDD Works Well
TDD shines when building business logic, utility classes, data transformations, and domain models. It works best when requirements are clear enough to express as test cases.

### When TDD Is Less Suited
TDD is harder to apply when you are exploring an unfamiliar API, prototyping UI layouts, or working with external systems that are difficult to simulate. In these situations, you might write the code first, then add tests afterward.

## Common Pitfalls
1. **Skipping the Red phase** — If you write the production code first, you lose the feedback loop. You can never be sure the test actually catches regressions because you never saw it fail.
2. **Over-engineering in the Green phase** — Writing a full implementation during the Green phase defeats the purpose. Write just enough to pass, then let additional tests drive the remaining logic.

## Best Practices
1. **Run tests after every change** — The cycle only works if you get rapid feedback. Keep your test suite fast so you can run it after every small step.
2. **Commit after each Green-Refactor pair** — Each passing cycle is a safe checkpoint. Committing frequently means you can always roll back to a known-good state.

## Summary
- TDD follows the Red-Green-Refactor cycle: write a failing test, make it pass, then clean up.
- The Red phase ensures your test is meaningful by confirming it fails first.
- The Green phase focuses on the simplest possible passing implementation.
- The Refactor phase improves code quality while tests guarantee correctness.
- TDD works best for business logic and domain models, and is less suited for exploratory or UI work.

## Code Examples

**The complete Red-Green-Refactor cycle for building an email validator with TDD**

```php
<?php
// The TDD cycle in action: building an EmailValidator

// STEP 1 - RED: Write a test that fails
// EmailValidator does not exist yet, so this test fails
class EmailValidatorTest extends TestCase
{
    public function testValidEmailReturnsTrue(): void
    {
        $validator = new EmailValidator();
        $this->assertTrue($validator->isValid('user@example.com'));
    }

    public function testInvalidEmailReturnsFalse(): void
    {
        $validator = new EmailValidator();
        $this->assertFalse($validator->isValid('not-an-email'));
    }
}

// STEP 2 - GREEN: Write the simplest code that passes
class EmailValidator
{
    public function isValid(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
}

// STEP 3 - REFACTOR: Improve without breaking tests
// (e.g., add stricter validation, extract constants, improve naming)
```


## Resources

- [PHPUnit Documentation - Writing Tests](https://docs.phpunit.de/en/12.0/writing-tests-for-phpunit.html) — Official PHPUnit guide on writing tests, the foundation of TDD
- [Martin Fowler - Test-Driven Development](https://martinfowler.com/bliki/TestDrivenDevelopment.html) — Martin Fowler's explanation of TDD principles and workflow

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*