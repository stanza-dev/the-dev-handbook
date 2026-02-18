---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-observer-pattern"
---

# Observer Pattern

The Observer pattern defines a one-to-many dependency between objects. When one object changes state, all its dependents are notified automatically.

## Basic Implementation

```php
<?php
interface Observer
{
    public function update(string $event, mixed $data): void;
}

interface Subject
{
    public function attach(Observer $observer): void;
    public function detach(Observer $observer): void;
    public function notify(string $event, mixed $data): void;
}

trait Observable
{
    private array $observers = [];
    
    public function attach(Observer $observer): void
    {
        $this->observers[] = $observer;
    }
    
    public function detach(Observer $observer): void
    {
        $this->observers = array_filter(
            $this->observers,
            fn($o) => $o !== $observer
        );
    }
    
    public function notify(string $event, mixed $data): void
    {
        foreach ($this->observers as $observer) {
            $observer->update($event, $data);
        }
    }
}
```

## Domain Event Example

```php
<?php
class User implements Subject
{
    use Observable;
    
    public function __construct(
        public readonly int $id,
        private string $email
    ) {}
    
    public function changeEmail(string $newEmail): void
    {
        $oldEmail = $this->email;
        $this->email = $newEmail;
        
        $this->notify('email_changed', [
            'user_id' => $this->id,
            'old_email' => $oldEmail,
            'new_email' => $newEmail,
        ]);
    }
}

// Observers
class EmailNotifier implements Observer
{
    public function update(string $event, mixed $data): void
    {
        if ($event === 'email_changed') {
            $this->sendConfirmationEmail($data['new_email']);
        }
    }
}

class AuditLogger implements Observer
{
    public function update(string $event, mixed $data): void
    {
        $this->log->info("Event: $event", $data);
    }
}

class CacheInvalidator implements Observer
{
    public function update(string $event, mixed $data): void
    {
        if ($event === 'email_changed') {
            $this->cache->delete("user:{$data['user_id']}");
        }
    }
}

// Usage
$user = new User(1, 'old@example.com');
$user->attach(new EmailNotifier());
$user->attach(new AuditLogger());
$user->attach(new CacheInvalidator());

$user->changeEmail('new@example.com');
// All observers notified automatically
```

## Event Dispatcher

```php
<?php
class EventDispatcher
{
    private array $listeners = [];
    
    public function listen(string $event, callable $listener): void
    {
        $this->listeners[$event][] = $listener;
    }
    
    public function dispatch(string $event, array $payload = []): void
    {
        foreach ($this->listeners[$event] ?? [] as $listener) {
            $listener($payload);
            
            // Stop propagation if listener returns false
            if ($listener($payload) === false) {
                break;
            }
        }
    }
}

// Usage
$dispatcher = new EventDispatcher();

$dispatcher->listen('user.created', function($data) {
    echo "Send welcome email to {$data['email']}\n";
});

$dispatcher->listen('user.created', function($data) {
    echo "Add to newsletter: {$data['email']}\n";
});

$dispatcher->dispatch('user.created', [
    'id' => 1,
    'email' => 'user@example.com',
]);
```

## PHP SPL Implementation

```php
<?php
class UserRegistration extends SplSubject
{
    private SplObjectStorage $observers;
    private array $data;
    
    public function __construct()
    {
        $this->observers = new SplObjectStorage();
    }
    
    public function attach(SplObserver $observer): void
    {
        $this->observers->attach($observer);
    }
    
    public function detach(SplObserver $observer): void
    {
        $this->observers->detach($observer);
    }
    
    public function notify(): void
    {
        foreach ($this->observers as $observer) {
            $observer->update($this);
        }
    }
    
    public function register(array $data): void
    {
        $this->data = $data;
        // Save user...
        $this->notify();
    }
    
    public function getData(): array
    {
        return $this->data;
    }
}
```

## Resources

- [SplObserver](https://www.php.net/manual/en/class.splobserver.php) — PHP SPL Observer interface

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*