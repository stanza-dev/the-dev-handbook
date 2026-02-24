---
source_course: "go-std-lib"
source_lesson: "go-std-lib-fuzzing"
---

# Fuzz Testing

## Introduction
Manual test cases only cover the scenarios you think of. Fuzz testing flips the script: Go's built-in fuzzer generates thousands of random inputs per second, probing your code for crashes, panics, and logic errors you never anticipated. Introduced in Go 1.18, fuzzing has become a standard tool for hardening parsers, validators, and any function that accepts untrusted input.

## Key Concepts
- **testing.F**: The type passed to fuzz test functions. It provides `f.Add()` for seeding the corpus and `f.Fuzz()` for defining the fuzz target.
- **Seed corpus**: A set of known-good inputs provided via `f.Add()` that the fuzzer uses as starting points for mutation.
- **Fuzz target**: The function passed to `f.Fuzz()` that receives randomly generated inputs and checks invariants.
- **Corpus directory**: The `testdata/fuzz/FuzzName/` directory where the fuzzer saves inputs that triggered failures, turning discovered bugs into permanent regression tests.

## Real World Context
You maintain a JSON configuration parser that has worked reliably for months. You write a fuzz test with a round-trip invariant: parse the input, serialize the result, and parse again. Within 10 seconds, the fuzzer discovers that a deeply nested object with escaped Unicode sequences causes a panic in your parser. The failing input is automatically saved to `testdata/fuzz/FuzzConfig/`, so every future `go test` run replays it as a regression test. You fix the bug and the fuzz test becomes a permanent safety net.

## Deep Dive
Fuzz test functions live in `_test.go` files and start with `Fuzz` followed by a capitalized name. They accept a single `*testing.F` parameter.

The basic structure has two parts: seeding the corpus and defining the fuzz target.

```go
func FuzzParse(f *testing.F) {
    // Add seed corpus
    f.Add("hello")
    f.Add("world")
    f.Add("")

    f.Fuzz(func(t *testing.T, input string) {
        result, err := Parse(input)
        if err != nil {
            return // Expected for invalid input
        }

        // Round-trip check
        encoded := Encode(result)
        if encoded != input {
            t.Errorf("round-trip failed: %q -> %v -> %q",
                input, result, encoded)
        }
    })
}
```

The `f.Add()` calls provide seed values that the fuzzer mutates. Each `f.Add()` argument must match the parameter types in `f.Fuzz()`. The fuzz target function receives a `*testing.T` (not `*testing.F`) so you use standard test assertions inside it.

Notice the `return` on error: invalid input is expected during fuzzing. Your fuzz target should only fail on logic violations (broken invariants), not on gracefully handled parse errors.

### Running Fuzz Tests

Fuzzing is not enabled by default. You must pass the `-fuzz` flag.

```bash
go test -fuzz=FuzzParse              # Run fuzzer indefinitely
go test -fuzz=FuzzParse -fuzztime=30s # Run for 30 seconds
go test -run=FuzzParse               # Run seeds only (no fuzzing)
```

The `-fuzz` flag starts the fuzzer, which runs continuously until it finds a failure or you stop it. The `-fuzztime` flag limits the duration. Running with `-run=FuzzParse` (no `-fuzz`) executes the seed corpus as a regular test, which is what CI should do.

### The Corpus

When the fuzzer discovers an input that causes a failure, it saves the input to `testdata/fuzz/FuzzParse/` in your project. Every subsequent `go test` run replays these saved inputs, ensuring the bug never regresses. You should commit the `testdata/fuzz/` directory to version control.

### Supported Parameter Types

The fuzz target can accept multiple parameters. Supported types include `string`, `[]byte`, `int`, `int8` through `int64`, `uint` through `uint64`, `float32`, `float64`, `rune`, and `bool`.

```go
func FuzzMultiParam(f *testing.F) {
    f.Add("test", 42, true)
    f.Fuzz(func(t *testing.T, s string, n int, flag bool) {
        // Fuzzer varies all three parameters independently
    })
}
```

Each call to `f.Add()` must provide values matching the parameter list exactly.

### Writing Effective Invariants

The best fuzz targets check properties that must hold for all inputs. Common patterns include round-trip consistency (parse then serialize), idempotency (applying an operation twice gives the same result), and no-panic guarantees.

```go
func FuzzReverse(f *testing.F) {
    f.Add("hello")
    f.Add("world")
    f.Fuzz(func(t *testing.T, s string) {
        rev := Reverse(s)
        doubleRev := Reverse(rev)
        if s != doubleRev {
            t.Errorf("Reverse(Reverse(%q)) = %q", s, doubleRev)
        }
    })
}
```

This test verifies that reversing a string twice returns the original. The fuzzer will find edge cases like multi-byte UTF-8 characters, empty strings, and strings with combining marks.

As of Go 1.26, fuzz tests can leverage `t.ArtifactDir()` inside the fuzz target to write debug output when investigating complex failures. The `testing/synctest` package (stable since Go 1.25) can be used alongside fuzz tests when your fuzz target exercises concurrent code, allowing deterministic control over goroutine scheduling even under random inputs.

## Common Pitfalls
1. **Treating parse errors as test failures** -- During fuzzing, most random inputs are invalid. If your fuzz target calls `t.Fatal` on every parse error, the fuzzer cannot explore interesting mutations. Return early on expected errors and only fail on invariant violations.
2. **Not committing the testdata/fuzz directory** -- Failing inputs saved by the fuzzer are your regression tests. If you do not commit them, CI never replays them and the same bug can return.
3. **Running fuzzing in CI without a time limit** -- Fuzzing runs indefinitely by default. Always use `-fuzztime` in CI pipelines (e.g., `-fuzztime=60s`) to prevent jobs from hanging.

## Best Practices
1. **Provide diverse seed corpus values** -- Include empty strings, Unicode characters, maximum-length inputs, and known edge cases. The fuzzer mutates seeds, so diverse starting points lead to better coverage.
2. **Check invariants, not specific outputs** -- Fuzz targets should verify properties like round-trip consistency, no-panic guarantees, and output bounds rather than comparing against a fixed expected value.
3. **Run fuzzing locally before merging** -- Spend a few minutes fuzzing locally with `go test -fuzz=. -fuzztime=2m` to catch low-hanging bugs before they enter the main branch.

## Summary
- Fuzz tests use `func FuzzXxx(f *testing.F)` with `f.Add()` for seeds and `f.Fuzz()` for the target.
- The fuzzer generates random inputs to find crashes and invariant violations you did not anticipate.
- Failing inputs are saved to `testdata/fuzz/` as permanent regression tests.
- Use `-fuzz` to start fuzzing and `-fuzztime` to limit duration; use `-run` without `-fuzz` in CI.
- Go 1.26 adds `t.ArtifactDir()` for debug output, and `testing/synctest` (stable since Go 1.25) helps fuzz concurrent code deterministically.

## Code Examples

**Fuzz Test Example**

```go
func FuzzReverse(f *testing.F) {
    f.Add("hello")
    f.Add("世界")
    
    f.Fuzz(func(t *testing.T, s string) {
        rev := Reverse(s)
        doubleRev := Reverse(rev)
        
        if s != doubleRev {
            t.Errorf("Reverse(Reverse(%q)) = %q", s, doubleRev)
        }
    })
}
```


## Resources

- [Go Fuzzing - Go Documentation](https://go.dev/doc/security/fuzz/) — Official Go documentation on fuzzing
- [Fuzzing Tutorial](https://go.dev/doc/tutorial/fuzz) — Official Go tutorial on writing fuzz tests

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*