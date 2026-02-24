---
source_course: "go-std-lib"
source_lesson: "go-std-lib-table-driven-tests"
---

# Table-Driven Tests

## Introduction
Table-driven tests are the idiomatic way to write tests in Go. Instead of writing a separate test function for every scenario, you define a slice of test cases and loop over them with `t.Run`. This lesson shows you how to structure table-driven tests that are easy to read, extend, and debug.

## Key Concepts
- **testing.T**: The type passed to every test function. It provides methods like `Error`, `Fatal`, and `Run` for reporting failures and organizing subtests.
- **Table-driven test**: A pattern where test cases are defined as a slice of structs, each containing inputs and expected outputs, then iterated with a loop.
- **Subtest**: A named test created with `t.Run` that can be filtered and run independently via `go test -run TestName/subtest`.

## Real World Context
Imagine you maintain a URL-parsing library. Every time a bug report arrives, you add one line to your test table instead of writing an entirely new function. Six months later your table has 40 rows covering every edge case anyone has ever hit, all in a single readable block. Table-driven tests also integrate naturally with CI dashboards because each `t.Run` subtest appears as its own pass/fail line.

## Deep Dive
Go has a built-in testing framework with zero external dependencies. Test files must end in `_test.go` and test functions must start with `Test` followed by a capitalized name, accepting a single `*testing.T` parameter.

Here is a complete table-driven test that exercises an `Add` function with several input combinations.

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name       string
        a, b, want int
    }{
        {"positive", 1, 2, 3},
        {"zero", 0, 0, 0},
        {"negative", -1, 1, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            if got != tt.want {
                t.Errorf("Add(%d, %d) = %d; want %d",
                    tt.a, tt.b, got, tt.want)
            }
        })
    }
}
```

Each struct in the slice represents one test scenario. The `t.Run` call wraps each case in a named subtest, so the output clearly identifies which case failed. You can run a single subtest by name with `go test -run TestAdd/negative`.

The key reporting methods on `testing.T` let you control how failures are handled.

```go
t.Error(args...)          // Log failure, continue test
t.Errorf(format, args...) // Formatted failure, continue test
t.Fatal(args...)          // Log failure, stop immediately
t.Skip(args...)           // Skip this test with a reason
```

Use `t.Error` when you want the rest of the test to keep running so you can see all failures at once. Use `t.Fatal` when a failure makes subsequent checks meaningless, for example when setup fails.

As of Go 1.26, `testing.T` also provides `t.ArtifactDir()`, which returns a unique per-test directory for writing output files such as golden snapshots or debug logs. The directory is automatically created and scoped to the running test, so parallel tests never collide. The `testing/synctest` package, which graduated from experimental in Go 1.26, provides utilities for deterministically testing concurrent code by controlling fake clocks and goroutine scheduling.

## Common Pitfalls
1. **Forgetting t.Run** -- Looping without `t.Run` means all cases share one test name and you cannot filter or parallelize individual cases.
2. **Capturing the loop variable in a closure** -- In Go versions before 1.22, the loop variable `tt` was shared across iterations. If you launch parallel subtests, capture `tt` with `tt := tt` inside the loop body, or upgrade to Go 1.22+ where the variable is scoped per iteration.
3. **Using t.Fatal inside a goroutine** -- `t.Fatal` calls `runtime.Goexit`, which only terminates the current goroutine. Calling it from a spawned goroutine will not stop the test. Use `t.Error` plus a return instead.

## Best Practices
1. **Name every test case** -- A descriptive `name` field makes failure output self-explanatory and lets you isolate a single case with `go test -run`.
2. **Keep the struct anonymous** -- Declaring the struct inline next to the test cases keeps everything in one place and avoids polluting the package namespace.
3. **Group related assertions** -- Within a single subtest, check all related properties before returning so you see the full picture on failure.

## Summary
- Go tests live in `_test.go` files; functions start with `Test` and accept `*testing.T`.
- Table-driven tests define cases as a slice of structs and iterate with `t.Run` for named subtests.
- Use `t.Error` to report and continue, `t.Fatal` to report and stop.
- Go 1.26 adds `t.ArtifactDir()` for per-test output directories.
- The `testing/synctest` package (stable since Go 1.25) helps test concurrent code deterministically.

## Code Examples

**Subtests with t.Run**

```go
func TestSplit(t *testing.T) {
    // ... define tests
    t.Run("subtest", func(t *testing.T) {
        // Subtests allow granular reporting
    })
}
```


## Resources

- [testing package - Go Documentation](https://pkg.go.dev/testing) — Official reference for testing.T, subtests, and test organization
- [Using Subtests and Sub-benchmarks](https://go.dev/blog/subtests) — Official Go blog post on table-driven tests and subtests

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*