---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-interface-segregation"
---

# Interface Segregation in Practice

## Introduction
The Interface Segregation Principle (ISP) states that clients should not be forced to depend on interfaces they do not use. This lesson explores practical techniques for breaking down fat interfaces into focused, composable contracts.

## Key Concepts
- **Fat Interface**: An interface with too many methods forcing implementors to provide unused functionality.
- **Segregated Interface**: A small, focused interface describing a single capability.
- **Interface Composition**: Combining small interfaces via multiple implementation or inheritance.
- **Role Interface**: An interface named after the role it plays for its clients.

## Real World Context
You are building a CMS with a `CrudRepository` interface having find, create, update, delete, search, paginate, and export methods. When you need a read-only report repository, it is forced to implement write methods with exceptions. This is a clear sign the interface is too fat.

## Deep Dive

### The Problem: Fat Interfaces

```php
<?php
interface CrudRepository
{
    public function find(int $id): ?object;
    public function findAll(): array;
    public function create(array $data): object;
    public function update(int $id, array $data): object;
    public function delete(int $id): void;
    public function search(string $query): array;
    public function export(): string;
}

class ReadOnlyRepository implements CrudRepository
{
    public function create(array $data): object
    {
        throw new BadMethodCallException('Read-only!');
    }
    // ... more unused methods throwing exceptions
}
```

This violates ISP because `ReadOnlyRepository` is forced to depend on methods it does not use.

### The Solution: Segregated Interfaces

```php
<?php
interface Readable
{
    public function find(int $id): ?object;
    public function findAll(): array;
}

interface Writable
{
    public function create(array $data): object;
    public function update(int $id, array $data): object;
    public function delete(int $id): void;
}

interface Searchable
{
    public function search(string $query): array;
}

interface Exportable
{
    public function export(): string;
}

class UserRepository implements Readable, Writable, Searchable
{
    // Only implements what it needs
}

class ReportRepository implements Readable, Exportable
{
    // Read and export only
}
```

Each class implements only the interfaces matching its actual capabilities.

### Notification System Example

```php
<?php
interface EmailNotifiable {
    public function getEmail(): string;
}

interface SmsNotifiable {
    public function getPhoneNumber(): string;
}

class User implements EmailNotifiable, SmsNotifiable
{
    public function getEmail(): string { return $this->email; }
    public function getPhoneNumber(): string { return $this->phone; }
}

class SystemAccount implements EmailNotifiable
{
    public function getEmail(): string { return 'admin@example.com'; }
}

class EmailSender
{
    public function send(EmailNotifiable $recipient, string $msg): void
    {
        mail($recipient->getEmail(), 'Notification', $msg);
    }
}
```

`EmailSender` only depends on `EmailNotifiable`, not SMS methods.

## Common Pitfalls
1. **Going too granular** — An interface per method creates type explosion. Group closely related methods.
2. **Ignoring ISP early** — Starting fat and planning to refactor later usually means it never happens.

## Best Practices
1. **Start small** — It is easier to combine small interfaces than to split large ones.
2. **Use intersection types** — PHP 8.1+ supports `Readable&Searchable` in parameter declarations.

## Summary
- Fat interfaces force implementors to provide methods they do not use.
- Segregated interfaces are small, focused, and composable.
- Each class should implement only the interfaces matching its capabilities.
- Small interfaces reduce coupling and simplify testing.

## Code Examples

**Segregated interfaces with intersection types**

```php
<?php
declare(strict_types=1);

interface Readable {
    public function find(int $id): ?object;
    public function findAll(): array;
}

interface Writable {
    public function create(array $data): object;
    public function delete(int $id): void;
}

interface Searchable {
    public function search(string $query): array;
}

class UserRepository implements Readable, Writable, Searchable {
    public function find(int $id): ?object { return null; }
    public function findAll(): array { return []; }
    public function create(array $data): object { return new stdClass(); }
    public function delete(int $id): void {}
    public function search(string $query): array { return []; }
}

class CacheRepository implements Readable {
    public function find(int $id): ?object { return null; }
    public function findAll(): array { return []; }
}

function loadAndSearch(Readable&Searchable $repo): array {
    return $repo->search('test');
}
?>
```


## Resources

- [Interface Segregation](https://www.php.net/manual/en/language.oop5.interfaces.php) — PHP interfaces documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*