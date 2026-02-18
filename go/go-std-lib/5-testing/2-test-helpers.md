---
source_course: "go-std-lib"
source_lesson: "go-std-lib-test-helpers"
---

# Test Organization

## Test Helpers

Mark helper functions with `t.Helper()` for better error reporting:

```go
func assertEqual(t *testing.T, got, want int) {
    t.Helper()  // Points errors to the caller
    if got != want {
        t.Errorf("got %d, want %d", got, want)
    }
}
```

## Setup & Teardown

```go
func TestWithSetup(t *testing.T) {
    // Setup
    db := setupTestDB(t)
    defer db.Close()  // Teardown
    
    // Test code...
}
```

## Cleanup Function

```go
func TestWithCleanup(t *testing.T) {
    f := createTempFile(t)
    t.Cleanup(func() {
        os.Remove(f.Name())
    })
    
    // t.Cleanup runs even if test fails
}
```

## TestMain

For package-level setup:

```go
func TestMain(m *testing.M) {
    // Setup
    setup()
    
    code := m.Run()  // Run all tests
    
    // Teardown
    teardown()
    
    os.Exit(code)
}
```

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


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*