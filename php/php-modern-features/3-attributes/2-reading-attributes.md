---
source_course: "php-modern-features"
source_lesson: "php-modern-features-reading-attributes"
---

# Reading Attributes with Reflection

## Introduction
Attributes are metadata only — they have no effect unless your code reads them. PHP's Reflection API provides the methods to discover and instantiate attributes at runtime, enabling attribute-driven frameworks and libraries.

## Key Concepts
- **ReflectionAttribute**: The object returned when querying attributes, containing the attribute class name and arguments.
- **`newInstance()`**: Creates an instance of the attribute class, running its constructor with the provided arguments.
- **Filtering**: You can query for specific attribute classes or use `IS_INSTANCEOF` to include subclasses.

## Real World Context
Frameworks like Symfony and Laravel use reflection to scan controller classes for `#[Route]` attributes at boot time, building routing tables automatically. ORM libraries scan entity classes for `#[Column]` attributes to build database schemas. Understanding attribute reflection is key to building and debugging these systems.

## Deep Dive

### Basic Attribute Reading

Read attributes from a class:

```php
<?php
#[Attribute]
class Route {
    public function __construct(
        public string $path,
        public string $method = 'GET'
    ) {}
}

#[Route('/home')]
class HomeController {}

$reflection = new ReflectionClass(HomeController::class);
$attributes = $reflection->getAttributes(Route::class);

foreach ($attributes as $attribute) {
    $route = $attribute->newInstance();
    echo "Path: " . $route->path;   // /home
    echo "Method: " . $route->method; // GET
}
```

The `getAttributes()` method returns an array of `ReflectionAttribute` objects. Calling `newInstance()` creates the actual attribute object.

### Reading Method Attributes

Scan methods for route definitions:

```php
<?php
class UserController {
    #[Route('/users', 'GET')]
    public function index(): array { return []; }
    
    #[Route('/users/{id}', 'GET')]
    public function show(int $id): array { return []; }
}

$reflection = new ReflectionClass(UserController::class);

foreach ($reflection->getMethods() as $method) {
    $attributes = $method->getAttributes(Route::class);
    
    foreach ($attributes as $attr) {
        $route = $attr->newInstance();
        echo "{$route->method} {$route->path} -> {$method->getName()}\n";
    }
}
// GET /users -> index
// GET /users/{id} -> show
```

This pattern is the core of every attribute-based routing system.

### Reading Property Attributes

Inspect properties for ORM-style column mappings:

```php
<?php
#[Attribute]
class Column {
    public function __construct(
        public string $name,
        public string $type = 'string'
    ) {}
}

class User {
    #[Column('user_id', 'integer')]
    public int $id;
    
    #[Column('user_name')]
    public string $name;
}

$reflection = new ReflectionClass(User::class);

foreach ($reflection->getProperties() as $property) {
    $columns = $property->getAttributes(Column::class);
    
    foreach ($columns as $attr) {
        $column = $attr->newInstance();
        echo "{$property->getName()} -> {$column->name} ({$column->type})\n";
    }
}
```

This enables declarative database mapping where the schema is defined alongside the code.

### Filtering Attributes

Filter by class or parent class:

```php
<?php
// Get only Route attributes
$routeAttrs = $reflection->getAttributes(Route::class);

// Get all attributes
$allAttrs = $reflection->getAttributes();

// Include subclasses of Validator
$validatorAttrs = $reflection->getAttributes(
    Validator::class,
    ReflectionAttribute::IS_INSTANCEOF
);
```

The `IS_INSTANCEOF` flag is useful when you have an attribute hierarchy and want to find all attributes that extend a base class.

## Common Pitfalls
1. **Calling `newInstance()` on invalid attributes** — If the attribute class does not exist or the constructor arguments are wrong, `newInstance()` throws an exception. Always handle this gracefully in framework code.
2. **Performance of repeated reflection** — Reflection is not free. Cache the results of attribute scanning during application bootstrap rather than scanning on every request.

## Best Practices
1. **Scan attributes once at boot time** — Build a lookup table (routing table, validator map) during application initialization and cache it, rather than using reflection on every request.
2. **Use `IS_INSTANCEOF` for attribute hierarchies** — If you have a base `Validator` attribute with subclasses like `Required` and `Email`, filter with `IS_INSTANCEOF` to find all validators at once.

## Summary
- Use `getAttributes()` on reflection objects to discover attributes.
- Call `newInstance()` to instantiate the attribute class with its constructor arguments.
- Filter by specific class or use `IS_INSTANCEOF` for attribute hierarchies.
- Cache attribute scanning results for performance.
- Attributes power declarative routing, validation, ORM mapping, and more.

## Code Examples

**Building a validator that reads attribute metadata via reflection to validate DTO properties**

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_PROPERTY)]
class Required {}

#[Attribute(Attribute::TARGET_PROPERTY)]
class Email {}

class Validator {
    public function validate(object $object): array {
        $errors = [];
        $reflection = new ReflectionClass($object);
        
        foreach ($reflection->getProperties() as $property) {
            $property->setAccessible(true);
            $value = $property->getValue($object);
            $name = $property->getName();
            
            if ($property->getAttributes(Required::class)) {
                if (empty($value)) {
                    $errors[$name][] = "$name is required";
                }
            }
            
            if ($property->getAttributes(Email::class)) {
                if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
                    $errors[$name][] = "$name must be a valid email";
                }
            }
        }
        
        return $errors;
    }
}

class UserDTO {
    #[Required]
    public string $name = '';
    
    #[Required]
    #[Email]
    public string $email = '';
}

$dto = new UserDTO();
$dto->name = '';
$dto->email = 'invalid';

$validator = new Validator();
$errors = $validator->validate($dto);
print_r($errors);
?>
```


## Resources

- [Reading Attributes](https://www.php.net/manual/en/language.attributes.reflection.php) — How to read attributes with Reflection API

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*