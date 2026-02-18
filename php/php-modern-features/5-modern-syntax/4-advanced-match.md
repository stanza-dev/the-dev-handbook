---
source_course: "php-modern-features"
source_lesson: "php-modern-features-advanced-match"
---

# Advanced Match Expression Patterns

The match expression goes beyond simple value matching. Let's explore advanced patterns.

## Combining Conditions with match(true)

```php
<?php
function classifyNumber(int $n): string
{
    return match(true) {
        $n < 0 => 'negative',
        $n === 0 => 'zero',
        $n > 0 && $n < 10 => 'small positive',
        $n >= 10 && $n < 100 => 'medium positive',
        default => 'large positive',
    };
}

echo classifyNumber(5);   // 'small positive'
echo classifyNumber(50);  // 'medium positive'
echo classifyNumber(-3);  // 'negative'
```

## Match with Complex Return Values

```php
<?php
enum HttpMethod { case GET; case POST; case PUT; case DELETE; }

function getRouteConfig(HttpMethod $method, string $resource): array
{
    return match([$method, $resource]) {
        [HttpMethod::GET, 'users'] => [
            'controller' => 'UserController',
            'action' => 'index',
            'middleware' => ['auth'],
        ],
        [HttpMethod::POST, 'users'] => [
            'controller' => 'UserController',
            'action' => 'store',
            'middleware' => ['auth', 'admin'],
        ],
        [HttpMethod::GET, 'posts'] => [
            'controller' => 'PostController',
            'action' => 'index',
            'middleware' => [],
        ],
        default => throw new NotFoundException(),
    };
}
```

## Match with Object Types

```php
<?php
interface Shape {}
class Circle implements Shape { public function __construct(public float $radius) {} }
class Rectangle implements Shape { public function __construct(public float $width, public float $height) {} }
class Triangle implements Shape { public function __construct(public float $base, public float $height) {} }

function calculateArea(Shape $shape): float
{
    return match($shape::class) {
        Circle::class => pi() * $shape->radius ** 2,
        Rectangle::class => $shape->width * $shape->height,
        Triangle::class => 0.5 * $shape->base * $shape->height,
        default => throw new InvalidArgumentException('Unknown shape'),
    };
}

$circle = new Circle(5);
echo calculateArea($circle);  // ~78.54
```

## Nested Match Expressions

```php
<?php
function getPricing(string $plan, bool $annual): array
{
    return match($plan) {
        'basic' => [
            'name' => 'Basic',
            'price' => match($annual) {
                true => 99,
                false => 12,
            },
            'period' => $annual ? 'year' : 'month',
        ],
        'pro' => [
            'name' => 'Professional',
            'price' => match($annual) {
                true => 299,
                false => 35,
            },
            'period' => $annual ? 'year' : 'month',
        ],
        default => throw new InvalidArgumentException("Unknown plan: $plan"),
    };
}
```

## Match with Callbacks

```php
<?php
$operation = 'uppercase';
$transformer = match($operation) {
    'uppercase' => fn($s) => strtoupper($s),
    'lowercase' => fn($s) => strtolower($s),
    'reverse' => fn($s) => strrev($s),
    'trim' => fn($s) => trim($s),
    default => fn($s) => $s,
};

echo $transformer('Hello');  // 'HELLO'
```

## Resources

- [Match Expression](https://www.php.net/manual/en/control-structures.match.php) — PHP match expression documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*