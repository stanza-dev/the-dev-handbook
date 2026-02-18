---
source_course: "go-web-services"
source_lesson: "go-web-services-sql-security"
---

# SQL Injection Prevention

**Always use parameterized queries!**

```go
// VULNERABLE - Never do this!
query := "SELECT * FROM users WHERE name = '" + input + "'"

// SAFE - Use placeholders
db.Query("SELECT * FROM users WHERE name = $1", input)
```

## Placeholder Syntax

*   PostgreSQL: `$1`, `$2`, `$3`
*   MySQL: `?`, `?`, `?`
*   SQLite: `?` or `$1`

## Nullable Fields

```go
type User struct {
    ID    int
    Name  string
    Email sql.NullString  // Handles NULL
}

// Check if valid
if user.Email.Valid {
    fmt.Println(user.Email.String)
}
```

## Context Cancellation

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

rows, err := db.QueryContext(ctx, "SELECT ...")
```

Cancelling the context will interrupt long-running queries.

## Code Examples

**Safe SQL Queries**

```go
// Safe: parameterized query
rows, err := db.Query(
    "SELECT * FROM users WHERE name = $1 AND status = $2",
    userName,
    "active",
)

// Also safe: prepared statement
stmt, _ := db.Prepare("SELECT * FROM users WHERE id = $1")
row := stmt.QueryRow(userID)
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*