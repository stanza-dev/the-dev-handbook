---
source_course: "php-api-development"
source_lesson: "php-api-development-pagination"
---

# Pagination & Filtering

Proper pagination and filtering are essential for API performance and usability.

## Offset Pagination

```php
<?php
class PaginatedResponse {
    public static function create(
        array $items,
        int $total,
        int $page,
        int $perPage
    ): array {
        return [
            'data' => $items,
            'meta' => [
                'current_page' => $page,
                'per_page' => $perPage,
                'total' => $total,
                'total_pages' => (int) ceil($total / $perPage),
            ],
            'links' => [
                'first' => "?page=1&per_page=$perPage",
                'last' => "?page=" . ceil($total / $perPage) . "&per_page=$perPage",
                'prev' => $page > 1 ? "?page=" . ($page - 1) . "&per_page=$perPage" : null,
                'next' => $page < ceil($total / $perPage) ? "?page=" . ($page + 1) . "&per_page=$perPage" : null,
            ],
        ];
    }
}

// Controller
class UserController extends Controller {
    public function index(): never {
        $page = max(1, (int) ($_GET['page'] ?? 1));
        $perPage = min(100, max(1, (int) ($_GET['per_page'] ?? 20)));
        $offset = ($page - 1) * $perPage;
        
        $total = $this->users->count();
        $users = $this->users->findAll($perPage, $offset);
        
        $this->json(PaginatedResponse::create($users, $total, $page, $perPage));
    }
}
```

## Cursor Pagination

```php
<?php
// Better for large datasets - no offset performance issues
class CursorPaginator {
    public function paginate(
        PDO $pdo,
        string $table,
        int $limit,
        ?string $cursor = null
    ): array {
        $sql = "SELECT * FROM $table";
        $params = [];
        
        if ($cursor) {
            $decodedCursor = json_decode(base64_decode($cursor), true);
            $sql .= ' WHERE id > :cursor_id';
            $params['cursor_id'] = $decodedCursor['id'];
        }
        
        $sql .= ' ORDER BY id ASC LIMIT :limit';
        
        $stmt = $pdo->prepare($sql);
        $stmt->bindValue(':limit', $limit + 1, PDO::PARAM_INT);
        foreach ($params as $key => $value) {
            $stmt->bindValue(":$key", $value);
        }
        $stmt->execute();
        
        $items = $stmt->fetchAll();
        $hasMore = count($items) > $limit;
        
        if ($hasMore) {
            array_pop($items);
        }
        
        $nextCursor = null;
        if ($hasMore && !empty($items)) {
            $lastItem = end($items);
            $nextCursor = base64_encode(json_encode(['id' => $lastItem['id']]));
        }
        
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

## Filtering

```php
<?php
class QueryFilter {
    private array $allowedFilters = [
        'status' => 'exact',
        'name' => 'like',
        'created_at' => 'date_range',
        'price' => 'range',
    ];
    
    public function apply(QueryBuilder $query, array $filters): QueryBuilder {
        foreach ($filters as $field => $value) {
            if (!isset($this->allowedFilters[$field])) {
                continue;
            }
            
            $type = $this->allowedFilters[$field];
            
            $query = match($type) {
                'exact' => $query->where($field, $value),
                'like' => $query->whereLike($field, "%$value%"),
                'range' => $this->applyRange($query, $field, $value),
                'date_range' => $this->applyDateRange($query, $field, $value),
                default => $query,
            };
        }
        
        return $query;
    }
    
    private function applyRange(QueryBuilder $query, string $field, array $value): QueryBuilder {
        if (isset($value['min'])) {
            $query = $query->where($field, $value['min'], '>=');
        }
        if (isset($value['max'])) {
            $query = $query->where($field, $value['max'], '<=');
        }
        return $query;
    }
}

// Usage: GET /products?status=active&name=widget&price[min]=10&price[max]=100
```

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*