---
source_course: "php-modern-features"
source_lesson: "php-modern-features-reading-attributes"
---

# Reading Attributes with Reflection

Attributes are meaningless without code that reads them. Use PHP's Reflection API to access attribute metadata.

## Basic Attribute Reading

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

// Read the attribute
$reflection = new ReflectionClass(HomeController::class);
$attributes = $reflection->getAttributes(Route::class);

foreach ($attributes as $attribute) {
    $route = $attribute->newInstance();
    echo "Path: " . $route->path;   // /home
    echo "Method: " . $route->method; // GET
}
```

## Reading Method Attributes

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
// Output:
// GET /users -> index
// GET /users/{id} -> show
```

## Reading Property Attributes

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

## Filtering Attributes

```php
<?php
// Get only attributes of specific class
$routeAttrs = $reflection->getAttributes(Route::class);

// Get all attributes
$allAttrs = $reflection->getAttributes();

// Filter by parent class (include subclasses)
$validatorAttrs = $reflection->getAttributes(
    Validator::class,
    ReflectionAttribute::IS_INSTANCEOF
);
```

## Complete Router Example

```php
<?php
function registerRoutes(string $controllerClass): array {
    $routes = [];
    $reflection = new ReflectionClass($controllerClass);
    
    foreach ($reflection->getMethods(ReflectionMethod::IS_PUBLIC) as $method) {
        foreach ($method->getAttributes(Route::class) as $attr) {
            $route = $attr->newInstance();
            $routes[] = [
                'path' => $route->path,
                'method' => $route->method,
                'handler' => [$controllerClass, $method->getName()]
            ];
        }
    }
    
    return $routes;
}
```

## Code Examples

**Building a validator using attributes and reflection**

```php
<?php
declare(strict_types=1);

// Attribute-based validator
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
            
            // Check Required
            if ($property->getAttributes(Required::class)) {
                if (empty($value)) {
                    $errors[$name][] = "$name is required";
                }
            }
            
            // Check Email
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