---
source_course: "php-databases"
source_lesson: "php-databases-relationships"
---

# Modeling Relationships in PHP

## Introduction

Relational databases express relationships through foreign keys, but PHP code needs to load and navigate those relationships efficiently. Understanding how to model one-to-one, one-to-many, and many-to-many relationships determines both data integrity and query performance.

## Key Concepts

- **One-to-Many**: A parent row has multiple child rows (e.g., user has many orders). The child table holds the foreign key.
- **Many-to-Many**: Both tables relate to each other through a junction (pivot) table (e.g., products and tags).
- **Foreign Key Constraint**: A database rule ensuring that a child row's foreign key references a valid parent row.

## Real World Context

A blog platform has users who write posts, posts that have tags, and comments that belong to both a user and a post. Modeling these relationships correctly in the schema and loading them efficiently in PHP is the difference between a fast application and one that chokes under load.

## Deep Dive

One-to-many is the most common relationship. The child table holds the foreign key:

```sql
CREATE TABLE users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);

CREATE TABLE posts (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT UNSIGNED NOT NULL,
    title VARCHAR(255) NOT NULL,
    body TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user (user_id)
);
```

Loading the relationship in PHP:

```php
<?php
// Load a user's posts
function getUserPosts(PDO $pdo, int $userId): array {
    $stmt = $pdo->prepare('SELECT * FROM posts WHERE user_id = :uid ORDER BY created_at DESC');
    $stmt->execute(['uid' => $userId]);
    return $stmt->fetchAll();
}
```

Many-to-many uses a junction table:

```sql
CREATE TABLE tags (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE post_tags (
    post_id INT UNSIGNED NOT NULL,
    tag_id INT UNSIGNED NOT NULL,
    PRIMARY KEY (post_id, tag_id),
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);
```

Querying many-to-many:

```php
<?php
// Get all tags for a post
function getPostTags(PDO $pdo, int $postId): array {
    $stmt = $pdo->prepare('
        SELECT t.* FROM tags t
        JOIN post_tags pt ON t.id = pt.tag_id
        WHERE pt.post_id = :pid
    ');
    $stmt->execute(['pid' => $postId]);
    return $stmt->fetchAll();
}

// Get all posts with a specific tag
function getPostsByTag(PDO $pdo, string $tagName): array {
    $stmt = $pdo->prepare('
        SELECT p.* FROM posts p
        JOIN post_tags pt ON p.id = pt.post_id
        JOIN tags t ON pt.tag_id = t.id
        WHERE t.name = :tag
    ');
    $stmt->execute(['tag' => $tagName]);
    return $stmt->fetchAll();
}
```

One-to-one uses a UNIQUE constraint on the foreign key:

```sql
CREATE TABLE user_profiles (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT UNSIGNED NOT NULL UNIQUE,  -- UNIQUE enforces one-to-one
    bio TEXT,
    avatar_url VARCHAR(500),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

Cascade options determine what happens when a parent is deleted:

```sql
-- ON DELETE CASCADE: delete children when parent is deleted
-- ON DELETE SET NULL: set foreign key to NULL
-- ON DELETE RESTRICT: prevent deletion if children exist
```

## Common Pitfalls

1. **Missing indexes on foreign key columns** — Without an index on the foreign key column, every JOIN or WHERE clause on that column requires a full table scan.
2. **Using ON DELETE CASCADE on financial data** — Deleting a user should not cascade-delete their order history. Use RESTRICT or SET NULL for audit-sensitive tables.

## Best Practices

1. **Always index foreign key columns** — MySQL creates them automatically for FOREIGN KEY constraints, but other databases may not.
2. **Choose CASCADE actions based on business rules** — CASCADE for dependent data (post comments), RESTRICT for important records (orders), SET NULL for optional relationships.

## Summary

- One-to-many places the foreign key on the child table; many-to-many uses a junction table.
- Always index foreign key columns and define constraints to enforce referential integrity.
- Choose ON DELETE behavior (CASCADE, SET NULL, RESTRICT) based on business requirements.
- Load relationships with JOINs or IN clauses rather than N+1 individual queries.

## Code Examples

**Many-to-many tag sync within a transaction**

```php
<?php
declare(strict_types=1);

// Attach and detach tags (many-to-many)
function syncPostTags(PDO $pdo, int $postId, array $tagIds): void {
    $pdo->beginTransaction();
    try {
        // Remove existing tags
        $pdo->prepare('DELETE FROM post_tags WHERE post_id = ?')
            ->execute([$postId]);
        
        // Attach new tags
        $stmt = $pdo->prepare(
            'INSERT INTO post_tags (post_id, tag_id) VALUES (?, ?)'
        );
        foreach ($tagIds as $tagId) {
            $stmt->execute([$postId, $tagId]);
        }
        
        $pdo->commit();
    } catch (Throwable $e) {
        $pdo->rollBack();
        throw $e;
    }
}
?>
```


## Resources

- [MySQL Foreign Keys](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html) — MySQL foreign key constraint reference

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*