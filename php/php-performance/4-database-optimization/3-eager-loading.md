---
source_course: "php-performance"
source_lesson: "php-performance-eager-loading"
---

# Eager Loading & Query Optimization

## Introduction
Eager loading fetches related data in bulk, avoiding the N+1 query problem that plagues many applications.

## Key Concepts
- **Eager Loading**: Fetching related data in a single query with JOIN or WHERE IN, instead of one query per related record.
- **PDO::FETCH_UNIQUE**: Indexes fetched results by a column value for O(1) lookup instead of O(n) array scanning.
- **Batch Fetching**: Loading related records in groups (e.g., `WHERE id IN (1,2,3,4,5)`) instead of individual queries.
- **Lazy Loading Trap**: ORMs that transparently load related data trigger N+1 queries unless you explicitly configure eager loading.

## Real World Context
The transition from N+1 to eager loading is often the single biggest performance improvement in PHP applications. A page loading 50 users with their orders can go from 51 queries (1 for users + 50 for orders) to 2 queries (1 for users + 1 for all orders), reducing database time from 500ms to 10ms.

## Deep Dive
### Intro

Eager loading fetches related data in bulk, avoiding the N+1 query problem that plagues many applications.

### The n+1 problem visualized

```php
<?php
// N+1 Problem: 1 query for posts + N queries for authors
$posts = $pdo->query('SELECT * FROM posts LIMIT 10')->fetchAll();

foreach ($posts as $post) {
    // Executes 10 times!
    $author = $pdo->prepare('SELECT * FROM users WHERE id = ?');
    $author->execute([$post['user_id']]);
    echo $post['title'] . ' by ' . $author->fetch()['name'];
}
// Total: 11 queries
```

### Solution: join query

```php
<?php
// Single query with JOIN
$sql = '
    SELECT posts.*, users.name as author_name
    FROM posts
    JOIN users ON posts.user_id = users.id
    LIMIT 10
';

$posts = $pdo->query($sql)->fetchAll();

foreach ($posts as $post) {
    echo $post['title'] . ' by ' . $post['author_name'];
}
// Total: 1 query
```

### Solution: batch loading

```php
<?php
class PostRepository
{
    public function getPostsWithAuthors(int $limit = 10): array
    {
        // Query 1: Get posts
        $posts = $this->pdo->query("SELECT * FROM posts LIMIT $limit")->fetchAll();
        
        if (empty($posts)) {
            return [];
        }
        
        // Extract unique user IDs
        $userIds = array_unique(array_column($posts, 'user_id'));
        $placeholders = implode(',', array_fill(0, count($userIds), '?'));
        
        // Query 2: Get all authors in one query
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id IN ($placeholders)");
        $stmt->execute($userIds);
        $users = $stmt->fetchAll(PDO::FETCH_UNIQUE);  // Index by ID
        
        // Combine data
        foreach ($posts as &$post) {
            $post['author'] = $users[$post['user_id']] ?? null;
        }
        
        return $posts;
    }
}
// Total: 2 queries (regardless of how many posts)
```

### Lazy loading with data mapper

```php
<?php
class LazyPost
{
    private ?User $author = null;
    private bool $authorLoaded = false;
    
    public function __construct(
        public int $id,
        public string $title,
        public int $userId,
        private UserRepository $userRepo
    ) {}
    
    public function getAuthor(): User
    {
        if (!$this->authorLoaded) {
            $this->author = $this->userRepo->find($this->userId);
            $this->authorLoaded = true;
        }
        return $this->author;
    }
    
    // Pre-load to avoid lazy loading
    public function setAuthor(User $user): void
    {
        $this->author = $user;
        $this->authorLoaded = true;
    }
}
```

### Query analysis with explain

```php
<?php
function analyzeQuery(PDO $pdo, string $sql): array
{
    $stmt = $pdo->query('EXPLAIN ' . $sql);
    $plan = $stmt->fetchAll(PDO::FETCH_ASSOC);
    
    $issues = [];
    
    foreach ($plan as $row) {
        // Full table scan
        if ($row['type'] === 'ALL' && $row['rows'] > 1000) {
            $issues[] = "Full table scan on {$row['table']} ({$row['rows']} rows)";
        }
        
        // No index used
        if (empty($row['key'])) {
            $issues[] = "No index used for {$row['table']}";
        }
        
        // Filesort
        if (str_contains($row['Extra'] ?? '', 'filesort')) {
            $issues[] = "Using filesort for {$row['table']}";
        }
        
        // Temporary table
        if (str_contains($row['Extra'] ?? '', 'temporary')) {
            $issues[] = "Using temporary table for {$row['table']}";
        }
    }
    
    return [
        'plan' => $plan,
        'issues' => $issues,
    ];
}
```

## Common Pitfalls
1. **Eager loading everything** — Loading all related data regardless of whether it's needed increases memory usage and query complexity. Only eager-load what the current page actually displays.
2. **Not using indexed result sets** — After fetching related records, using `array_filter()` or loops to match them is O(n×m). Use `PDO::FETCH_UNIQUE` or `array_column()` for O(1) lookups.

## Best Practices
1. **Use `FETCH_UNIQUE` for indexed lookups** — When you need to match related records to parent records, index the results by the foreign key for instant lookups.
2. **Monitor query counts per page** — Set up a development middleware that counts queries and alerts when the count exceeds a threshold (e.g., 20 queries per page).

## Summary
- Eager loading with JOIN or WHERE IN eliminates the N+1 query problem.
- Use `PDO::FETCH_UNIQUE` to index result sets for O(1) lookups.
- Monitor query counts per page to catch N+1 regressions early.

## Resources

- [MySQL EXPLAIN](https://dev.mysql.com/doc/refman/8.4/en/explain.html) — Understanding MySQL query execution plans

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*