---
source_course: "php-performance"
source_lesson: "php-performance-eager-loading"
---

# Eager Loading & Query Optimization

Eager loading fetches related data in bulk, avoiding the N+1 query problem that plagues many applications.

## The N+1 Problem Visualized

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

## Solution: JOIN Query

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

## Solution: Batch Loading

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

## Lazy Loading with Data Mapper

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

## Query Analysis with EXPLAIN

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

## Resources

- [MySQL EXPLAIN](https://dev.mysql.com/doc/refman/8.0/en/explain.html) — Understanding MySQL query execution plans

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*