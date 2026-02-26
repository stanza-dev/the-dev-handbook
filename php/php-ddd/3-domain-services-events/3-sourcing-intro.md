---
source_course: "php-ddd"
source_lesson: "php-ddd-event-sourcing-intro"
---

# Introduction to Event Sourcing

## Introduction

Event Sourcing is an architectural pattern where application state is stored as a sequence of events rather than as a snapshot of the current state. Instead of persisting the latest version of an entity, every change is captured as an immutable event and replayed to reconstruct state on demand.

## Key Concepts

- **Event Store** - a persistent, append-only log of all domain events
- **Event Replay** - reconstructing an aggregate's state by replaying its event history
- **Snapshot** - a cached point-in-time representation of aggregate state to avoid replaying all events
- **Projection** - a read-optimized view built by processing events
- **Temporal Query** - querying what the state looked like at a specific point in time

## Real World Context

Event Sourcing is used in financial systems, audit-heavy domains, and anywhere a complete history of changes is valuable. Banks, for example, record every transaction rather than only storing the current balance, enabling full auditability and temporal queries.

## Deep Dive

### Why Event Sourcing?

Traditional persistence overwrites state on every save, losing the history of how that state was reached. Event Sourcing keeps a complete, immutable audit trail that can be replayed, analyzed, or projected into different read models.

### A Simple Event-Sourced Aggregate

```php
<?php
declare(strict_types=1);

namespace Domain\Wallet;

final class Wallet
{
    private WalletId $id;
    private Money $balance;
    /** @var DomainEvent[] */
    private array $recordedEvents = [];
    private int $version = 0;

    private function __construct(WalletId $id)
    {
        $this->id = $id;
        $this->balance = Money::zero();
    }

    public static function open(WalletId $id, Money $initialDeposit): self
    {
        $wallet = new self($id);
        $wallet->apply(new WalletOpened($id, $initialDeposit));
        return $wallet;
    }

    public function deposit(Money $amount): void
    {
        if ($amount->isNegativeOrZero()) {
            throw new \InvalidArgumentException('Deposit amount must be positive.');
        }
        $this->apply(new MoneyDeposited($this->id, $amount));
    }

    public function withdraw(Money $amount): void
    {
        if ($amount->isGreaterThan($this->balance)) {
            throw InsufficientFunds::forWithdrawal($this->id, $amount, $this->balance);
        }
        $this->apply(new MoneyWithdrawn($this->id, $amount));
    }

    // --- Event application ---

    private function apply(DomainEvent $event): void
    {
        $this->mutate($event);
        $this->recordedEvents[] = $event;
    }

    private function mutate(DomainEvent $event): void
    {
        match (true) {
            $event instanceof WalletOpened => $this->whenWalletOpened($event),
            $event instanceof MoneyDeposited => $this->whenMoneyDeposited($event),
            $event instanceof MoneyWithdrawn => $this->whenMoneyWithdrawn($event),
            default => throw new \RuntimeException('Unknown event: ' . get_class($event)),
        };
        $this->version++;
    }

    private function whenWalletOpened(WalletOpened $event): void
    {
        $this->balance = $event->initialDeposit;
    }

    private function whenMoneyDeposited(MoneyDeposited $event): void
    {
        $this->balance = $this->balance->add($event->amount);
    }

    private function whenMoneyWithdrawn(MoneyWithdrawn $event): void
    {
        $this->balance = $this->balance->subtract($event->amount);
    }

    // --- Reconstitution from history ---

    public static function fromHistory(array $events): self
    {
        $first = $events[0] ?? throw new \RuntimeException('No events to replay.');
        $wallet = new self($first->walletId);

        foreach ($events as $event) {
            $wallet->mutate($event);
        }

        return $wallet;
    }

    public function pullRecordedEvents(): array
    {
        $events = $this->recordedEvents;
        $this->recordedEvents = [];
        return $events;
    }

    public function id(): WalletId { return $this->id; }
    public function balance(): Money { return $this->balance; }
    public function version(): int { return $this->version; }
}
```

### The Event Store

```php
<?php
declare(strict_types=1);

namespace Infrastructure\EventStore;

final class SqlEventStore implements EventStore
{
    public function __construct(private \PDO $pdo) {}

    public function append(string $aggregateId, array $events, int $expectedVersion): void
    {
        $this->pdo->beginTransaction();

        try {
            $currentVersion = $this->currentVersion($aggregateId);
            if ($currentVersion !== $expectedVersion) {
                throw new ConcurrencyException(
                    "Expected version {$expectedVersion}, got {$currentVersion}."
                );
            }

            $stmt = $this->pdo->prepare('
                INSERT INTO event_store (aggregate_id, version, event_type, payload, occurred_at)
                VALUES (?, ?, ?, ?, ?)
            ');

            $version = $expectedVersion;
            foreach ($events as $event) {
                $version++;
                $stmt->execute([
                    $aggregateId,
                    $version,
                    get_class($event),
                    json_encode($event, JSON_THROW_ON_ERROR),
                    $event->occurredAt()->format('Y-m-d H:i:s.u'),
                ]);
            }

            $this->pdo->commit();
        } catch (\Exception $e) {
            $this->pdo->rollBack();
            throw $e;
        }
    }

    public function loadEvents(string $aggregateId): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT event_type, payload FROM event_store WHERE aggregate_id = ? ORDER BY version ASC'
        );
        $stmt->execute([$aggregateId]);

        return array_map(
            fn(array $row) => $this->deserialize($row['event_type'], $row['payload']),
            $stmt->fetchAll(\PDO::FETCH_ASSOC)
        );
    }

    private function currentVersion(string $aggregateId): int
    {
        $stmt = $this->pdo->prepare(
            'SELECT COALESCE(MAX(version), 0) FROM event_store WHERE aggregate_id = ?'
        );
        $stmt->execute([$aggregateId]);
        return (int) $stmt->fetchColumn();
    }
}
```

### Snapshots for Performance

When an aggregate has thousands of events, replaying them all becomes expensive. Snapshots cache the state at a point in time so that only events after the snapshot need replaying.

```php
<?php
final class SnapshotAwareWalletRepository implements WalletRepository
{
    public function __construct(
        private EventStore $eventStore,
        private SnapshotStore $snapshots,
        private int $snapshotFrequency = 100,
    ) {}

    public function find(WalletId $id): ?Wallet
    {
        $snapshot = $this->snapshots->load($id->toString());

        if ($snapshot) {
            $events = $this->eventStore->loadEventsSince(
                $id->toString(),
                $snapshot->version()
            );
            return Wallet::fromSnapshot($snapshot, $events);
        }

        $events = $this->eventStore->loadEvents($id->toString());
        return empty($events) ? null : Wallet::fromHistory($events);
    }

    public function save(Wallet $wallet): void
    {
        $events = $wallet->pullRecordedEvents();
        $this->eventStore->append($wallet->id()->toString(), $events, $wallet->version() - count($events));

        if ($wallet->version() % $this->snapshotFrequency === 0) {
            $this->snapshots->save($wallet->id()->toString(), $wallet->snapshot());
        }
    }
}
```

## Common Pitfalls

1. **Replaying all events without snapshots** - For long-lived aggregates, event replay becomes slow without periodic snapshots to short-circuit the process.
2. **Modifying past events** - Events are immutable facts. If the business logic changes, create new event types or compensating events instead of altering history.
3. **Confusing Event Sourcing with Event-Driven Architecture** - Event Sourcing is about persistence; event-driven architecture is about communication. They complement each other but are independent patterns.

## Best Practices

1. **Use snapshots for aggregates with many events** - Set a snapshot frequency (e.g., every 100 events) to keep reconstitution fast.
2. **Version your events** - Event schemas evolve over time. Use upcasters to transform old event formats into the current schema during replay.
3. **Build projections for reads** - Do not query the event store directly for read models. Instead, project events into denormalized read-optimized views.

## Summary

- Event Sourcing stores every state change as an immutable event instead of overwriting state
- Aggregates are reconstructed by replaying their event history
- Snapshots improve performance by caching state at intervals
- Projections build read-optimized views from the event stream
- Event Sourcing provides a complete audit trail and enables temporal queries

## Resources

- [Event Sourcing Pattern](https://martinfowler.com/eaaDev/EventSourcing.html) — Martin Fowler's introduction to Event Sourcing

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*