---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-interface-segregation"
---

# Interface Segregation in Practice

The Interface Segregation Principle (ISP) states that clients should not be forced to depend on interfaces they don't use.

## The Problem: Fat Interfaces

```php
<?php
// BAD: Fat interface forces unnecessary implementations
interface CrudRepository
{
    public function find(int $id): ?object;
    public function findAll(): array;
    public function create(array $data): object;
    public function update(int $id, array $data): object;
    public function delete(int $id): void;
    public function search(string $query): array;
    public function paginate(int $page, int $perPage): array;
    public function export(): string;
    public function import(string $data): void;
}

// ReadOnlyRepository forced to implement write methods!
class ReadOnlyRepository implements CrudRepository
{
    public function create(array $data): object
    {
        throw new BadMethodCallException('Read-only!');  // Ugly!
    }
    
    // ... more unused methods
}
```

## The Solution: Segregated Interfaces

```php
<?php
// GOOD: Small, focused interfaces
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

interface Paginatable
{
    public function paginate(int $page, int $perPage): array;
}

interface Exportable
{
    public function export(): string;
}

interface Importable
{
    public function import(string $data): void;
}

// Compose only what you need
class UserRepository implements Readable, Writable, Searchable
{
    // Only implements what it needs
}

class ReportRepository implements Readable, Exportable
{
    // Read and export only
}

class CacheRepository implements Readable
{
    // Read-only makes sense
}
```

## Real-World Example: Notification System

```php
<?php
// Segregated notification interfaces
interface Notifiable
{
    public function getNotificationId(): string;
}

interface EmailNotifiable extends Notifiable
{
    public function getEmail(): string;
    public function getEmailName(): string;
}

interface SmsNotifiable extends Notifiable
{
    public function getPhoneNumber(): string;
}

interface PushNotifiable extends Notifiable
{
    public function getDeviceTokens(): array;
}

// User implements all notification methods
class User implements EmailNotifiable, SmsNotifiable, PushNotifiable
{
    public function getNotificationId(): string
    {
        return (string) $this->id;
    }
    
    public function getEmail(): string { return $this->email; }
    public function getEmailName(): string { return $this->name; }
    public function getPhoneNumber(): string { return $this->phone; }
    public function getDeviceTokens(): array { return $this->devices; }
}

// System account only needs email
class SystemAccount implements EmailNotifiable
{
    public function getNotificationId(): string { return 'system'; }
    public function getEmail(): string { return 'admin@example.com'; }
    public function getEmailName(): string { return 'System'; }
}

// Type-hint exactly what you need
class EmailSender
{
    public function send(EmailNotifiable $recipient, string $message): void
    {
        // Only uses email methods
        mail($recipient->getEmail(), 'Notification', $message);
    }
}

class SmsSender
{
    public function send(SmsNotifiable $recipient, string $message): void
    {
        // Only uses phone methods
        $this->twilioClient->send($recipient->getPhoneNumber(), $message);
    }
}
```

## Benefits

1. **Reduced coupling**: Classes only depend on methods they use
2. **Easier testing**: Smaller interfaces = simpler mocks
3. **Better organization**: Clear purpose for each interface
4. **Flexibility**: Compose interfaces as needed

## Resources

- [Interface Segregation](https://www.php.net/manual/en/language.oop5.interfaces.php) — PHP interfaces documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*