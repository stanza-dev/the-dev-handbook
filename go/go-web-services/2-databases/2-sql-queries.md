---
source_course: "go-web-services"
source_lesson: "go-web-services-sql-queries"
---

# Query Types

## Single Row

```go
var user User
err := db.QueryRow("SELECT id, name FROM users WHERE id = $1", id).
    Scan(&user.ID, &user.Name)
```

## Multiple Rows

```go
rows, err := db.Query("SELECT id, name FROM users WHERE active = $1", true)
if err != nil {
    return err
}
defer rows.Close()

var users []User
for rows.Next() {
    var u User
    if err := rows.Scan(&u.ID, &u.Name); err != nil {
        return err
    }
    users = append(users, u)
}
return rows.Err()  // Check for iteration errors
```

## Prepared Statements

```go
stmt, err := db.Prepare("SELECT id, name FROM users WHERE id = $1")
if err != nil {
    return err
}
defer stmt.Close()

// Reuse stmt for multiple queries
for _, id := range ids {
    var user User
    stmt.QueryRow(id).Scan(&user.ID, &user.Name)
}
```

## Code Examples

**Query Multiple Rows**

```go
func getUsers(db *sql.DB) ([]User, error) {
    rows, err := db.Query("SELECT id, name, email FROM users")
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    var users []User
    for rows.Next() {
        var u User
        if err := rows.Scan(&u.ID, &u.Name, &u.Email); err != nil {
            return nil, err
        }
        users = append(users, u)
    }
    return users, rows.Err()
}
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*