---
source_course: "php-api-development"
source_lesson: "php-api-development-versioning-strategies"
---

# API Versioning Strategies

API versioning allows you to evolve your API while maintaining backward compatibility.

## URL Path Versioning

```php
<?php
// Most common approach
// /api/v1/users
// /api/v2/users

$router->group('/api/v1', function(Router $r) {
    $r->get('/users', [UserControllerV1::class, 'index']);
    $r->get('/users/{id}', [UserControllerV1::class, 'show']);
});

$router->group('/api/v2', function(Router $r) {
    $r->get('/users', [UserControllerV2::class, 'index']);
    $r->get('/users/{id}', [UserControllerV2::class, 'show']);
});
```

## Header Versioning

```php
<?php
// Accept: application/vnd.myapi.v1+json
// X-API-Version: 1

function getApiVersion(): int {
    // Check Accept header
    $accept = $_SERVER['HTTP_ACCEPT'] ?? '';
    if (preg_match('/vnd\.myapi\.v(\d+)/', $accept, $matches)) {
        return (int) $matches[1];
    }
    
    // Check custom header
    $version = $_SERVER['HTTP_X_API_VERSION'] ?? '1';
    return (int) $version;
}

// Route to correct version
$version = getApiVersion();
$controllerClass = match($version) {
    1 => UserControllerV1::class,
    2 => UserControllerV2::class,
    default => throw new BadRequestException('Unsupported API version'),
};
```

## Query Parameter Versioning

```php
<?php
// /api/users?version=2

$version = (int) ($_GET['version'] ?? 1);
```

## Version Abstraction Layer

```php
<?php
interface UserRepositoryInterface {
    public function findAll(): array;
    public function find(int $id): ?User;
}

// V1: Returns basic user data
class UserControllerV1 extends Controller {
    public function index(): never {
        $users = $this->users->findAll();
        
        $this->json([
            'data' => array_map(fn($u) => [
                'id' => $u->id,
                'name' => $u->name,
                'email' => $u->email,
            ], $users)
        ]);
    }
}

// V2: Returns extended data with profile
class UserControllerV2 extends Controller {
    public function index(): never {
        $users = $this->users->findAllWithProfile();
        
        $this->json([
            'data' => array_map(fn($u) => [
                'id' => $u->id,
                'name' => $u->name,
                'email' => $u->email,
                'profile' => [
                    'avatar' => $u->profile?->avatar,
                    'bio' => $u->profile?->bio,
                ],
                'created_at' => $u->createdAt->format('c'),
            ], $users),
            'meta' => [
                'total' => count($users),
            ]
        ]);
    }
}
```

## Deprecation Handling

```php
<?php
class DeprecationMiddleware {
    private array $deprecated = [
        '/api/v1/users' => '2025-01-01',
        '/api/v1/products' => '2025-06-01',
    ];
    
    public function handle(): void {
        $path = $_SERVER['REQUEST_URI'];
        
        foreach ($this->deprecated as $pattern => $sunsetDate) {
            if (str_starts_with($path, $pattern)) {
                header('Deprecation: true');
                header('Sunset: ' . date('D, d M Y H:i:s T', strtotime($sunsetDate)));
                header('Link: </api/v2/users>; rel="successor-version"');
                break;
            }
        }
    }
}
```

---

> 📘 *This lesson is part of the [RESTful API Development with PHP](https://stanza.dev/courses/php-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*