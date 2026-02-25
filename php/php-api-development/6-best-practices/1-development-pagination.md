---
source_course: "php-api-development"
source_lesson: "php-api-development-pagination"
---

# Pagination Strategies

## Introduction
Returning thousands of records in a single API response is a recipe for slow load times and out-of-memory errors. Pagination breaks large result sets into manageable pages, improving both performance and user experience. Choosing the right pagination strategy depends on your dataset size and access patterns.

## Key Concepts
- **Offset Pagination**: Skipping a fixed number of rows and returning the next batch, controlled by `page` and `per_page` parameters.
- **Cursor Pagination**: Using an opaque pointer (cursor) to the last item returned, fetching the next batch after that point.
- **Link Headers**: HTTP `Link` headers that provide URLs for the next, previous, first, and last pages, following RFC 8288.
- **Page Metadata**: Response envelope fields like `total`, `current_page`, and `total_pages` that help clients navigate the result set.

## Real World Context
Every production API needs pagination. GitHub uses Link headers for navigation, Stripe uses cursor-based pagination for large collections, and most CRUD APIs start with offset pagination for simplicity. Choosing the wrong strategy for a large dataset can turn a 50ms query into a 5-second timeout.

## Deep Dive

### Offset Pagination

Offset pagination is the simplest approach. The client sends a page number and items per page, and the server calculates the SQL `OFFSET` and `LIMIT`.

Here is a complete offset pagination implementation with metadata:

```php
<?php
class OffsetPaginator
{
    public function paginate(
        PDO $pdo,
        string $table,
        int $page = 1,
        int $perPage = 20
    ): array {
        $page = max(1, $page);
        $perPage = min(100, max(1, $perPage));
        $offset = ($page - 1) * $perPage;

        // Count total records
        $countStmt = $pdo->query("SELECT COUNT(*) FROM {$table}");
        $total = (int) $countStmt->fetchColumn();
        $totalPages = (int) ceil($total / $perPage);

        // Fetch page of results
        $stmt = $pdo->prepare(
            "SELECT * FROM {$table} ORDER BY id ASC LIMIT :limit OFFSET :offset"
        );
        $stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
        $stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
        $stmt->execute();

        return [
            'data' => $stmt->fetchAll(PDO::FETCH_ASSOC),
            'meta' => [
                'current_page' => $page,
                'per_page' => $perPage,
                'total' => $total,
                'total_pages' => $totalPages,
            ],
        ];
    }
}
```

Offset pagination is easy to implement and lets clients jump to any page. However, it has a performance problem: as the offset grows, the database must scan and discard all preceding rows.

### Cursor Pagination

Cursor pagination solves the offset performance problem by using a pointer to the last item instead of counting rows to skip. This is essential for large datasets.

Here is a cursor-based paginator that uses PHP 8.5's `array_first()` and `array_last()` functions for cleaner cursor handling:

```php
<?php
class CursorPaginator
{
    public function paginate(
        PDO $pdo,
        string $table,
        int $limit = 20,
        ?string $cursor = null
    ): array {
        $limit = min(100, max(1, $limit));
        $params = [];

        $sql = "SELECT * FROM {$table}";
        if ($cursor !== null) {
            $decoded = json_decode(base64_decode($cursor), true);
            $sql .= ' WHERE id > :after_id';
            $params[':after_id'] = $decoded['id'];
        }
        $sql .= ' ORDER BY id ASC LIMIT :fetch_limit';

        $stmt = $pdo->prepare($sql);
        $stmt->bindValue(':fetch_limit', $limit + 1, PDO::PARAM_INT);
        foreach ($params as $key => $value) {
            $stmt->bindValue($key, $value);
        }
        $stmt->execute();
        $items = $stmt->fetchAll(PDO::FETCH_ASSOC);

        $hasMore = count($items) > $limit;
        if ($hasMore) {
            array_pop($items);
        }

        // PHP 8.5: array_last() returns the last element cleanly
        $lastItem = array_last($items);
        $nextCursor = $hasMore && $lastItem
            ? base64_encode(json_encode(['id' => $lastItem['id']]))
            : null;

        // PHP 8.5: array_first() grabs the first element
        $firstItem = array_first($items);

        return [
            'data' => $items,
            'meta' => [
                'has_more' => $hasMore,
                'next_cursor' => $nextCursor,
            ],
        ];
    }
}
```

Cursor pagination fetches one extra row (`$limit + 1`) to determine if more data exists. The `array_last()` function from PHP 8.5 extracts the last element without modifying the array, making cursor construction cleaner than the traditional `end()` approach.

### Adding Link Headers

Link headers follow RFC 8288 and provide machine-readable navigation URLs. Many API clients and frameworks parse these headers automatically.

Here is how to add Link headers to your paginated response:

```php
<?php
function addPaginationLinkHeaders(
    string $baseUrl,
    int $currentPage,
    int $totalPages,
    int $perPage
): void {
    $links = [];

    if ($currentPage > 1) {
        $links[] = "<{$baseUrl}?page=1&per_page={$perPage}>; rel=\"first\"";
        $prev = $currentPage - 1;
        $links[] = "<{$baseUrl}?page={$prev}&per_page={$perPage}>; rel=\"prev\"";
    }

    if ($currentPage < $totalPages) {
        $next = $currentPage + 1;
        $links[] = "<{$baseUrl}?page={$next}&per_page={$perPage}>; rel=\"next\"";
        $links[] = "<{$baseUrl}?page={$totalPages}&per_page={$perPage}>; rel=\"last\"";
    }

    if (!empty($links)) {
        header('Link: ' . implode(', ', $links));
    }
}

// Usage in a controller
addPaginationLinkHeaders('/api/v1/products', $page, $totalPages, $perPage);
```

The Link header contains comma-separated entries with `rel` attributes indicating each link's purpose. Clients use `rel="next"` to fetch subsequent pages without constructing URLs themselves.

### Comparing the Two Approaches

| Feature | Offset | Cursor |
|---------|--------|--------|
| Jump to page N | Yes | No |
| Performance at scale | Degrades with offset | Constant |
| Consistent with inserts | Rows can shift | Stable |
| Implementation complexity | Simple | Moderate |
| Best for | Small datasets, admin UIs | Large datasets, feeds |

## Common Pitfalls
1. **No upper bound on per_page** — Without capping the `per_page` parameter, a client can request `?per_page=1000000` and crash your server. Always enforce a maximum (e.g., 100).
2. **Using offset pagination on large tables** — `OFFSET 500000` forces the database to scan half a million rows before returning results. Switch to cursor pagination once your dataset exceeds a few thousand rows.

## Best Practices
1. **Default to sensible values** — Set `page=1` and `per_page=20` as defaults. Clamp `per_page` between 1 and 100 to prevent abuse while giving clients flexibility.
2. **Include navigation metadata** — Always return `total`, `current_page`, and `total_pages` (for offset) or `has_more` and `next_cursor` (for cursor) so clients know how to navigate.

## Summary
- Offset pagination is simple but degrades on large datasets because the database must skip rows.
- Cursor pagination uses a pointer to the last item and maintains constant performance regardless of position.
- PHP 8.5's `array_first()` and `array_last()` simplify cursor extraction without array mutation.
- Link headers (RFC 8288) provide machine-readable navigation URLs for both pagination styles.
- Always cap `per_page` with an upper bound to prevent resource exhaustion.

## Code Examples

**Cursor-based pagination using PHP 8.5's array_last() for clean cursor extraction — fetches one extra row to detect if more data exists**

```php
<?php
// Cursor pagination with PHP 8.5's array_last()
class CursorPaginator
{
    public function paginate(
        PDO $pdo,
        string $table,
        int $limit = 20,
        ?string $cursor = null
    ): array {
        $sql = "SELECT * FROM {$table}";
        $params = [];

        if ($cursor !== null) {
            $decoded = json_decode(base64_decode($cursor), true);
            $sql .= ' WHERE id > :after_id';
            $params[':after_id'] = $decoded['id'];
        }

        $sql .= ' ORDER BY id ASC LIMIT ' . ($limit + 1);

        $stmt = $pdo->prepare($sql);
        foreach ($params as $k => $v) {
            $stmt->bindValue($k, $v);
        }
        $stmt->execute();
        $items = $stmt->fetchAll(PDO::FETCH_ASSOC);

        $hasMore = count($items) > $limit;
        if ($hasMore) {
            array_pop($items);
        }

        // PHP 8.5: array_last() returns last element without mutation
        $lastItem = array_last($items);
        $nextCursor = $hasMore && $lastItem
            ? base64_encode(json_encode(['id' => $lastItem['id']]))
            : null;

        return [
            'data' => $items,
            'meta' => ['has_more' => $hasMore, 'next_cursor' => $nextCursor],
        ];
    }
}
```


## Resources

- [PHP PDO Prepared Statements](https://www.php.net/manual/en/pdo.prepared-statements.php) — Official PHP documentation for PDO prepared statements used in pagination queries

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*