---
source_course: "go-std-lib"
source_lesson: "go-std-lib-test-helpers"
---

# Test Helpers & Setup

## Introduction
As your test suite grows, you will find yourself repeating setup logic, assertion helpers, and teardown code across many test functions. Go provides `t.Helper()`, `t.Cleanup()`, and `TestMain` to keep your tests DRY and maintainable without reaching for a third-party framework.

## Key Concepts
- **t.Helper()**: Marks a function as a test helper. When the helper calls `t.Errorf`, the error message points to the caller of the helper rather than the helper itself, making failures easier to locate.
- **t.Cleanup()**: Registers a function to run after the test completes, even if the test fails or panics. Cleanup functions run in LIFO (last registered, first run) order.
- **TestMain**: A special function `func TestMain(m *testing.M)` that gives you control over the entire test lifecycle of a package, including global setup and teardown.
- **t.ArtifactDir()**: New in Go 1.26, returns a unique directory path scoped to the current test for writing output artifacts like logs or snapshots.

## Real World Context
You are building a service that interacts with a database. Each integration test needs a fresh database connection, seed data, and cleanup afterward. Without helpers, every test function would repeat 10 lines of boilerplate. By extracting a `setupTestDB(t)` helper that uses `t.Helper()` for error attribution and `t.Cleanup()` for automatic teardown, you reduce each test to its essential logic. When a test fails, the error points to the test function that called the helper, not to the helper itself.

## Deep Dive

### Test Helpers with t.Helper()

When you write reusable assertion functions, call `t.Helper()` at the top so error messages reference the caller's file and line number.

```go
func assertEqual(t *testing.T, got, want int) {
    t.Helper() // Points errors to the caller
    if got != want {
        t.Errorf("got %d, want %d", got, want)
    }
}
```

Without `t.Helper()`, a failure would point to the `t.Errorf` line inside `assertEqual`, which is useless when the function is called from 20 different tests. With it, the failure points to the specific test line that called `assertEqual`.

### Setup and Teardown with defer

The simplest pattern for resource management in tests is the standard `defer` statement.

```go
func TestWithSetup(t *testing.T) {
    db := setupTestDB(t)
    defer db.Close() // Teardown

    // Test code using db...
}
```

This works well for single resources, but `defer` runs when the enclosing function returns, which can be too late for subtests.

### t.Cleanup() for Guaranteed Teardown

`t.Cleanup` is more powerful than `defer` because it is scoped to the test (or subtest), not the function. It runs after the test and all its subtests complete, even if the test panics.

```go
func TestWithCleanup(t *testing.T) {
    f := createTempFile(t)
    t.Cleanup(func() {
        os.Remove(f.Name())
    })

    // t.Cleanup runs even if test fails or panics
}
```

Multiple `t.Cleanup` calls are allowed; they execute in reverse registration order (LIFO), mirroring the behavior of `defer`.

### Combining Helpers with Cleanup

The most common pattern combines `t.Helper()` with `t.Cleanup()` in a factory function that returns a ready-to-use resource.

```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatal(err)
    }
    t.Cleanup(func() { db.Close() })
    return db
}
```

The caller gets a clean database and never has to worry about closing it. The helper handles errors, and the cleanup is automatic. As of Go 1.26, you can also use `t.ArtifactDir()` inside helpers to write test-specific output files to a directory that is automatically scoped per test.

### TestMain for Package-Level Setup

When you need setup that runs once for the entire package (e.g., starting a Docker container, loading fixtures), use `TestMain`.

```go
func TestMain(m *testing.M) {
    // Setup
    setup()

    code := m.Run() // Run all tests in the package

    // Teardown
    teardown()

    os.Exit(code)
}
```

`m.Run()` executes all `Test*` functions in the package and returns the exit code. The `os.Exit(code)` call at the end ensures the process reports the correct exit status. Only one `TestMain` is allowed per package.

## Common Pitfalls
1. **Forgetting t.Helper() in assertion functions** -- Without it, every failure points to the same line inside the helper rather than the test that triggered it. This makes debugging multiple failures nearly impossible.
2. **Using defer instead of t.Cleanup in subtests** -- `defer` runs when the outer function returns, not when the subtest finishes. If you create resources per subtest, use `t.Cleanup` so cleanup happens at the right scope.
3. **Calling os.Exit directly in TestMain without m.Run** -- Skipping `m.Run()` means no tests actually execute. Always call `m.Run()` and pass its result to `os.Exit`.

## Best Practices
1. **Extract setup into helper functions that accept *testing.T** -- This lets you use `t.Helper()`, `t.Fatal()`, and `t.Cleanup()` inside the helper, keeping test functions focused on assertions.
2. **Use t.Cleanup over defer for test resources** -- `t.Cleanup` is scoped to the test lifecycle, handles subtests correctly, and runs even on panic.
3. **Keep TestMain minimal** -- Only use it for truly global setup (database containers, expensive fixtures). Per-test setup belongs in helpers.

## Summary
- `t.Helper()` fixes error attribution so failures point to the calling test, not the helper function.
- `t.Cleanup()` registers teardown logic that runs after the test (and its subtests) complete, even on failure or panic.
- Combine `t.Helper()` and `t.Cleanup()` in factory helpers for clean, zero-boilerplate resource management.
- `TestMain(m *testing.M)` provides package-level setup and teardown around all tests.
- Go 1.26 adds `t.ArtifactDir()` for per-test output directories, useful in helpers that generate debug artifacts.

## Code Examples

**Test Helper with Cleanup**

```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatal(err)
    }
    t.Cleanup(func() { db.Close() })
    return db
}
```


## Resources

- [testing package - Go Documentation](https://pkg.go.dev/testing) — Official Go documentation for test helpers, cleanup, and TestMain
- [Add a test - Go Tutorial](https://go.dev/doc/tutorial/add-a-test) — Official Go tutorial on writing and organizing tests

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*