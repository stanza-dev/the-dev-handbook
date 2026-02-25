---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-reflection-api"
---

# Reflection API Deep Dive

## Introduction
Reflection allows you to inspect and manipulate classes, methods, and properties at runtime. It is the foundation for frameworks, ORMs, and dependency injection containers.

## Key Concepts
- **ReflectionClass**: Inspects class structure, methods, properties, and attributes.
- **ReflectionMethod**: Inspects method signatures, parameters, and return types.
- **ReflectionProperty**: Inspects property types, visibility, and values.
- **Attributes**: PHP 8 attributes can be read via Reflection for metadata-driven patterns.

## Real World Context
Dependency injection containers like Symfony's or Laravel's use Reflection to auto-wire dependencies. They inspect constructor parameters, resolve their types, and recursively create all needed objects. Understanding Reflection helps you build or debug these systems.

## Deep Dive

### Inspecting Classes

```php
<?php
class User {
    private string $id;
    public string $name;
    protected string $email;

    public function __construct(string $name, string $email) {
        $this->id = uniqid();
        $this->name = $name;
        $this->email = $email;
    }

    public function getEmail(): string {
        return $this->email;
    }
}

$reflection = new ReflectionClass(User::class);
echo $reflection->getName();         // User
echo $reflection->isInstantiable();  // true
echo $reflection->isFinal();         // false
```

ReflectionClass provides complete introspection of any class.

### Inspecting Properties

```php
<?php
$properties = $reflection->getProperties();

foreach ($properties as $property) {
    echo $property->getName();             // 'id', 'name', 'email'
    echo $property->getType()->getName();  // 'string'
    echo $property->isPublic();            // false, true, false
}

// Access private properties
$user = new User('John', 'john@example.com');
$idProperty = $reflection->getProperty('id');
// Note: setAccessible(true) is deprecated in PHP 8.5 and has no
// effect since PHP 8.1. All properties are accessible via Reflection.
echo $idProperty->getValue($user); // The private ID value
```

Since PHP 8.1, all properties are accessible via Reflection regardless of visibility. The `setAccessible(true)` call is deprecated in PHP 8.5 as it no longer has any effect.

### Inspecting Methods

```php
<?php
$methods = $reflection->getMethods(ReflectionMethod::IS_PUBLIC);

foreach ($methods as $method) {
    echo $method->getName();

    foreach ($method->getParameters() as $param) {
        echo $param->getName();
        echo $param->getType()?->getName();
        echo $param->isOptional();
    }

    echo $method->getReturnType()?->getName();
}
```

Parameter introspection is how DI containers know what dependencies to inject.

### Auto-Wiring Dependencies

```php
<?php
class SimpleContainer {
    private array $bindings = [];

    public function bind(string $abstract, string $concrete): void {
        $this->bindings[$abstract] = $concrete;
    }

    public function make(string $class): object {
        $concrete = $this->bindings[$class] ?? $class;
        $reflection = new ReflectionClass($concrete);

        if (!$reflection->isInstantiable()) {
            throw new Exception("Cannot instantiate $concrete");
        }

        $constructor = $reflection->getConstructor();
        if ($constructor === null) {
            return new $concrete();
        }

        $dependencies = [];
        foreach ($constructor->getParameters() as $param) {
            $type = $param->getType();
            if ($type === null || $type->isBuiltin()) {
                if ($param->isDefaultValueAvailable()) {
                    $dependencies[] = $param->getDefaultValue();
                } else {
                    throw new Exception("Cannot resolve: {$param->getName()}");
                }
            } else {
                $dependencies[] = $this->make($type->getName());
            }
        }

        return $reflection->newInstanceArgs($dependencies);
    }
}

$container = new SimpleContainer();
$container->bind(LoggerInterface::class, FileLogger::class);
$userService = $container->make(UserService::class);
```

The container inspects constructor parameters and recursively resolves dependencies.

### Attribute-Based Systems

```php
<?php
#[Attribute(Attribute::TARGET_PROPERTY)]
class Column {
    public function __construct(
        public string $name,
        public string $type = 'string'
    ) {}
}

class ORM {
    public function getColumns(string $class): array {
        $reflection = new ReflectionClass($class);
        $columns = [];

        foreach ($reflection->getProperties() as $property) {
            $attributes = $property->getAttributes(Column::class);
            if (!empty($attributes)) {
                $column = $attributes[0]->newInstance();
                $columns[$property->getName()] = $column;
            }
        }

        return $columns;
    }
}
```

Reflection reads PHP attributes to build metadata-driven systems like ORMs and validators.

## Common Pitfalls
1. **Using setAccessible(true) in PHP 8.5** — This method is deprecated and has no effect since PHP 8.1. Remove these calls.
2. **Performance in hot paths** — Reflection is relatively slow. Cache results when used in production code.

## Best Practices
1. **Cache reflection results** — Create the ReflectionClass once and reuse it rather than reflecting on every call.
2. **Use attributes over annotations** — PHP 8 native attributes are type-safe and faster than docblock annotations.

## Summary
- Reflection inspects classes, methods, and properties at runtime.
- Since PHP 8.1, all properties are accessible via Reflection; `setAccessible()` is deprecated in PHP 8.5.
- DI containers use Reflection to auto-wire constructor dependencies.
- PHP 8 attributes enable metadata-driven patterns through Reflection.

## Code Examples

**Attribute-based validation with Reflection**

```php
<?php
declare(strict_types=1);

#[Attribute(Attribute::TARGET_PROPERTY)]
class Validate {
    public function __construct(
        public string $rule,
        public string $message = ''
    ) {}
}

class UserDTO {
    #[Validate('required', 'Name is required')]
    public string $name;

    #[Validate('email', 'Invalid email')]
    public string $email;

    #[Validate('min:8', 'Password too short')]
    public string $password;
}

class Validator {
    public function validate(object $obj): array {
        $errors = [];
        $reflection = new ReflectionClass($obj);

        foreach ($reflection->getProperties() as $prop) {
            $attrs = $prop->getAttributes(Validate::class);
            foreach ($attrs as $attr) {
                $validate = $attr->newInstance();
                $value = $prop->getValue($obj);
                if ($validate->rule === 'required' && empty($value)) {
                    $errors[] = $validate->message;
                }
            }
        }
        return $errors;
    }
}
?>
```


## Resources

- [Reflection](https://www.php.net/manual/en/book.reflection.php) — PHP Reflection API documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*