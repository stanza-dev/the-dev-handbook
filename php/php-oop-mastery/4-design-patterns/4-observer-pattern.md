---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-observer-pattern"
---

# Observer Pattern

## Introduction
The Observer pattern defines a one-to-many dependency between objects. When one object changes state, all its dependents are notified automatically. This is the foundation of event-driven architectures.

## Key Concepts
- **Subject**: The object being observed, which maintains a list of observers.
- **Observer**: An object that wants to be notified of changes to the subject.
- **Event**: The specific change or action that triggers notification.
- **Loose Coupling**: The subject does not know the details of its observers.

## Real World Context
When a user registers on your platform, multiple things need to happen: send a welcome email, create an audit log entry, invalidate a cache, and add the user to a newsletter. The Observer pattern lets each action be an independent observer, so adding new actions requires no changes to the registration code.

## Deep Dive

### Basic Implementation

```php
<?php
interface Observer {
    public function update(string $event, mixed $data): void;
}

interface Subject {
    public function attach(Observer $observer): void;
    public function detach(Observer $observer): void;
    public function notify(string $event, mixed $data): void;
}

trait Observable {
    private array $observers = [];

    public function attach(Observer $observer): void {
        $this->observers[] = $observer;
    }

    public function detach(Observer $observer): void {
        $this->observers = array_filter(
            $this->observers,
            fn($o) => $o !== $observer
        );
    }

    public function notify(string $event, mixed $data): void {
        foreach ($this->observers as $observer) {
            $observer->update($event, $data);
        }
    }
}
```

The `Observable` trait provides the notification mechanism that any class can use.

### Domain Event Example

```php
<?php
class User implements Subject {
    use Observable;

    public function __construct(
        public readonly int $id,
        private string $email
    ) {}

    public function changeEmail(string $newEmail): void {
        $oldEmail = $this->email;
        $this->email = $newEmail;
        $this->notify('email_changed', [
            'user_id' => $this->id,
            'old_email' => $oldEmail,
            'new_email' => $newEmail,
        ]);
    }
}

class EmailNotifier implements Observer {
    public function update(string $event, mixed $data): void {
        if ($event === 'email_changed') {
            echo "Confirmation sent to {$data['new_email']}\n";
        }
    }
}

class AuditLogger implements Observer {
    public function update(string $event, mixed $data): void {
        echo "Audit: $event " . json_encode($data) . "\n";
    }
}

$user = new User(1, 'old@example.com');
$user->attach(new EmailNotifier());
$user->attach(new AuditLogger());
$user->changeEmail('new@example.com');
```

Each observer reacts independently. Adding new observers requires no changes to the User class.

### Event Dispatcher

```php
<?php
class EventDispatcher {
    private array $listeners = [];

    public function listen(string $event, callable $listener): void {
        $this->listeners[$event][] = $listener;
    }

    public function dispatch(string $event, array $payload = []): void {
        foreach ($this->listeners[$event] ?? [] as $listener) {
            $listener($payload);
        }
    }
}

$dispatcher = new EventDispatcher();
$dispatcher->listen('user.created', function($data) {
    echo "Welcome email to {$data['email']}\n";
});
$dispatcher->listen('user.created', function($data) {
    echo "Newsletter signup: {$data['email']}\n";
});
$dispatcher->dispatch('user.created', ['email' => 'user@example.com']);
```

An Event Dispatcher is a more flexible variation where listeners are registered as callables.

## Common Pitfalls
1. **Circular notifications** — Observer A changes subject, which notifies Observer B, which changes subject again. Add guards.
2. **Order dependency** — If observers must run in a specific order, the pattern may not be appropriate.

## Best Practices
1. **Use an Event Dispatcher** — For complex applications, a centralized dispatcher is cleaner than per-object observers.
2. **Keep observers lightweight** — Observers should do minimal work. For heavy tasks, dispatch to a queue.

## Summary
- Observer pattern enables one-to-many notifications without tight coupling.
- The subject does not know observer details, enabling independent evolution.
- Event dispatchers provide a centralized, flexible variation.
- Keep observers lightweight and watch for circular notifications.

## Code Examples

**Event dispatcher for order processing**

```php
<?php
declare(strict_types=1);

class EventDispatcher {
    private array $listeners = [];

    public function listen(string $event, callable $listener): void {
        $this->listeners[$event][] = $listener;
    }

    public function dispatch(string $event, array $payload = []): void {
        foreach ($this->listeners[$event] ?? [] as $listener) {
            $listener($payload);
        }
    }
}

$dispatcher = new EventDispatcher();

$dispatcher->listen('order.placed', function($data) {
    echo "Sending confirmation to {$data['email']}\n";
});

$dispatcher->listen('order.placed', function($data) {
    echo "Updating inventory for order {$data['id']}\n";
});

$dispatcher->listen('order.placed', function($data) {
    echo "Notifying warehouse about order {$data['id']}\n";
});

$dispatcher->dispatch('order.placed', [
    'id' => 42,
    'email' => 'customer@example.com',
]);
?>
```


## Resources

- [SplObserver](https://www.php.net/manual/en/class.splobserver.php) — PHP SPL Observer interface

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*