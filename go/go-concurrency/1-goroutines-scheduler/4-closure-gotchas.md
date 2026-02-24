---
source_course: "go-concurrency"
source_lesson: "go-concurrency-closure-gotchas"
---

# Closure Gotchas in Goroutines

## Introduction
One of the most common bugs in Go concurrency involves closures capturing loop variables by reference. Understanding this pitfall — and how Go 1.22 fixed it — is essential for writing correct concurrent code.

## Key Concepts
- **Closure:** A function that captures variables from its enclosing scope by reference.
- **Loop Variable Capture Bug:** In pre-Go 1.22, all iterations of a `for` loop share the same variable, so goroutines launched in the loop may all see the final value.
- **Per-Iteration Variable (Go 1.22+):** Each loop iteration creates a new variable, eliminating the capture bug.

## Real World Context
This bug has bitten nearly every Go developer at some point. Before Go 1.22, code reviews and linters like `go vet` flagged loop variable captures, and the standard workaround was to shadow the variable or pass it as a function argument. Even with the Go 1.22 fix, understanding the problem matters for maintaining legacy codebases.

## Deep Dive

### The Problem (Pre-Go 1.22)

```go
for i := 0; i < 5; i++ {
    go func() {
        fmt.Println(i)  // All might print 5!
    }()
}
```

All goroutines share the same `i` variable. By the time they execute, the loop has finished and `i` is 5.

### Solution 1: Pass as Argument

```go
for i := 0; i < 5; i++ {
    go func(n int) {
        fmt.Println(n)  // Correct: 0, 1, 2, 3, 4
    }(i)
}
```

### Solution 2: Shadow Variable

```go
for i := 0; i < 5; i++ {
    i := i  // Shadow with new variable
    go func() {
        fmt.Println(i)
    }()
}
```

### Go 1.22+ Fix
Go 1.22 changed loop variable semantics. Each iteration creates a new variable, fixing this issue automatically. But understanding the problem remains important for legacy code and for understanding closure semantics.

## Common Pitfalls
1. **Capturing range variables in goroutines** — Even with Go 1.22, capturing variables in closures that outlive their scope can cause subtle bugs if you are not aware of the semantics.
2. **Forgetting to shadow in pre-1.22 code** — When maintaining older codebases, always shadow or pass loop variables to goroutines.

## Best Practices
1. **Use the argument-passing pattern** — Even in Go 1.22+, passing values as arguments makes intent explicit and is universally understood.
2. **Run `go vet` regularly** — It detects loop variable capture issues and other common mistakes.

## Summary
- Pre-Go 1.22: closures in loops capture the variable by reference, seeing the final value.
- Fix: pass the variable as an argument or shadow it with `i := i`.
- Go 1.22+ creates a new variable per iteration, fixing the issue at the language level.
- Always run `go vet` to detect capture bugs.

## Code Examples

**Safe loop variable capture — shadowing the variable ensures each goroutine gets its own copy**

```go
// Correct pattern for all Go versions
for _, url := range urls {
    url := url  // Shadow: create a new variable per iteration
    go func() {
        fetch(url)
    }()
}
```


## Resources

- [Go Wiki: Common Mistakes — Using goroutines on loop iterator variables](https://go.dev/wiki/CommonMistakes) — Official documentation of the loop variable capture pitfall

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*