---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-reflection-api"
---

# Reflection API Deep Dive

Reflection allows you to inspect and manipulate classes, methods, and properties at runtime. It's the foundation for frameworks, ORMs, and dependency injection containers.

## Inspecting Classes

```php
<?php
class User
{
    private string $id;
    public string $name;
    protected string $email;
    
    public function __construct(string $name, string $email)
    {
        $this->id = uniqid();
        $this->name = $name;
        $this->email = $email;
    }
    
    public function getEmail(): string
    {
        return $this->email;
    }
}

$reflection = new ReflectionClass(User::class);

// Get class info
echo $reflection->getName();           // User
echo $reflection->getShortName();      // User (without namespace)
echo $reflection->isInstantiable();    // true
echo $reflection->isFinal();           // false
```

## Inspecting Properties

```php
<?php
$properties = $reflection->getProperties();

foreach ($properties as $property) {
    echo $property->getName();               // 'id', 'name', 'email'
    echo $property->getType()->getName();    // 'string'
    echo $property->isPublic();              // false, true, false
    echo $property->isPrivate();             // true, false, false
}

// Access private properties
$user = new User('John', 'john@example.com');
$idProperty = $reflection->getProperty('id');
$idProperty->setAccessible(true);  // Bypass visibility
echo $idProperty->getValue($user); // The private ID value
```

## Inspecting Methods

```php
<?php
$methods = $reflection->getMethods(ReflectionMethod::IS_PUBLIC);

foreach ($methods as $method) {
    echo $method->getName();
    
    // Get parameters
    foreach ($method->getParameters() as $param) {
        echo $param->getName();
        echo $param->getType()?->getName();
        echo $param->isOptional();
        echo $param->hasDefaultValue() ? $param->getDefaultValue() : 'none';
    }
    
    // Get return type
    echo $method->getReturnType()?->getName();
}
```

## Auto-Wiring Dependencies

```php
<?php
class SimpleContainer
{
    private array $bindings = [];
    
    public function bind(string $abstract, string $concrete): void
    {
        $this->bindings[$abstract] = $concrete;
    }
    
    public function make(string $class): object
    {
        // Check for binding
        $concrete = $this->bindings[$class] ?? $class;
        
        $reflection = new ReflectionClass($concrete);
        
        if (!$reflection->isInstantiable()) {
            throw new Exception("Cannot instantiate $concrete");
        }
        
        $constructor = $reflection->getConstructor();
        
        // No constructor = no dependencies
        if ($constructor === null) {
            return new $concrete();
        }
        
        // Resolve constructor dependencies
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
                // Recursively resolve class dependencies
                $dependencies[] = $this->make($type->getName());
            }
        }
        
        return $reflection->newInstanceArgs($dependencies);
    }
}

// Usage
$container = new SimpleContainer();
$container->bind(LoggerInterface::class, FileLogger::class);

$userService = $container->make(UserService::class);
// Automatically injects FileLogger!
```

## Creating Attribute-Based Systems

```php
<?php
#[Attribute(Attribute::TARGET_PROPERTY)]
class Column
{
    public function __construct(
        public string $name,
        public string $type = 'string'
    ) {}
}

class ORM
{
    public function getColumns(string $class): array
    {
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

## Resources

- [Reflection](https://www.php.net/manual/en/book.reflection.php) — PHP Reflection API documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*