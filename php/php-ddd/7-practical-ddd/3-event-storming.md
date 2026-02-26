---
source_course: "php-ddd"
source_lesson: "php-ddd-event-storming"
---

# Event Storming for Discovery

## Introduction

Event Storming is a collaborative workshop technique for rapidly exploring complex business domains. It brings together developers and domain experts to discover domain events, commands, aggregates, and bounded contexts through visual modeling on a large surface using sticky notes.

## Key Concepts

- **Domain Event** (orange sticky) - something that happened in the business, written in past tense
- **Command** (blue sticky) - an action that triggers a domain event
- **Aggregate** (yellow sticky) - the entity or cluster of entities that processes a command
- **Policy** (lilac sticky) - an automated reaction to an event that triggers another command
- **Read Model** (green sticky) - data needed by a user or system to make a decision

## Real World Context

Before writing any code, teams use Event Storming workshops to map an entire business process in hours rather than weeks. A team building an order management system can discover all the events from order creation to fulfillment, identify the aggregates responsible for each transition, and reveal bounded contexts that might not be obvious from requirements documents alone.

## Deep Dive

### Workshop Setup

Event Storming requires a large modeling space (a long wall), plenty of sticky notes in specific colors, and participants from both the development team and the business side. A facilitator guides the process but does not dictate the model.

### Phase 1: Chaotic Exploration

Everyone writes domain events on orange sticky notes and places them on the wall in rough chronological order. There is no discussion of correctness at this stage; the goal is to get everything out.

```
[OrderPlaced] → [PaymentReceived] → [OrderConfirmed] → [ItemsPicked] → [OrderShipped] → [OrderDelivered]
                  ↘ [PaymentFailed] → [OrderCancelled]
```

### Phase 2: Enforce the Timeline

The group arranges events in chronological order, identifies duplicates, and resolves conflicts. Hotspots (marked with red stickies) flag areas of confusion or disagreement.

### Phase 3: Add Commands and Actors

For each event, the group identifies what command triggered it and who or what issued that command.

```
Actor: Customer
Command: [Place Order] → Event: [OrderPlaced]

Actor: Payment Gateway
Command: [Process Payment] → Event: [PaymentReceived]

Actor: Warehouse Staff
Command: [Pick Items] → Event: [ItemsPicked]
```

### Phase 4: Identify Aggregates

Aggregates are the entities responsible for processing commands and producing events. They enforce business rules.

```
Aggregate: Order
  - handles [Place Order] → produces [OrderPlaced]
  - handles [Confirm Order] → produces [OrderConfirmed]
  - handles [Cancel Order] → produces [OrderCancelled]

Aggregate: Payment
  - handles [Process Payment] → produces [PaymentReceived] or [PaymentFailed]

Aggregate: Shipment
  - handles [Ship Order] → produces [OrderShipped]
```

### Phase 5: Identify Policies

Policies are automated reactions: when a certain event occurs, a command is automatically triggered.

```
Policy: "When [PaymentReceived], then [Confirm Order]"
Policy: "When [PaymentFailed] three times, then [Cancel Order]"
Policy: "When [OrderConfirmed], then [Reserve Inventory]"
```

### Phase 6: Discover Bounded Contexts

By examining clusters of related aggregates, events, and commands, the group identifies bounded contexts: areas of the domain that have their own model and ubiquitous language.

```
┌──────────────────────────┐  ┌──────────────────────────┐
│    Ordering Context       │  │    Fulfillment Context    │
│                          │  │                          │
│  Aggregates:             │  │  Aggregates:             │
│  - Order                 │  │  - Shipment              │
│  - Cart                  │  │  - Inventory             │
│                          │  │                          │
│  Events:                 │  │  Events:                 │
│  - OrderPlaced           │  │  - ItemsPicked           │
│  - OrderConfirmed        │  │  - OrderShipped          │
│  - OrderCancelled        │  │  - OrderDelivered        │
└──────────────────────────┘  └──────────────────────────┘

┌──────────────────────────┐  ┌──────────────────────────┐
│    Payment Context        │  │    Notification Context   │
│                          │  │                          │
│  Aggregates:             │  │  Policies:               │
│  - Payment               │  │  - Send email on         │
│                          │  │    OrderConfirmed        │
│  Events:                 │  │  - Send SMS on           │
│  - PaymentReceived       │  │    OrderShipped          │
│  - PaymentFailed         │  │                          │
└──────────────────────────┘  └──────────────────────────┘
```

### From Event Storming to Code

The artifacts from an Event Storming session translate directly to DDD building blocks.

```php
<?php
// Domain Events discovered in the workshop
final class OrderPlaced extends BaseDomainEvent { /* ... */ }
final class PaymentReceived extends BaseDomainEvent { /* ... */ }
final class OrderConfirmed extends BaseDomainEvent { /* ... */ }

// Commands identified during the workshop
final readonly class PlaceOrderCommand { /* ... */ }
final readonly class ProcessPaymentCommand { /* ... */ }

// Aggregates that emerged from clustering
final class Order { /* handles PlaceOrder, ConfirmOrder, CancelOrder */ }
final class Payment { /* handles ProcessPayment, RefundPayment */ }

// Policies that connect events to commands
final class ConfirmOrderOnPaymentReceived
{
    public function __invoke(PaymentReceived $event): void
    {
        $this->commandBus->dispatch(
            new ConfirmOrderCommand($event->orderId())
        );
    }
}
```

## Common Pitfalls

1. **Not inviting domain experts** - Event Storming without business stakeholders produces a technical model that misses real-world nuances and domain language.
2. **Jumping to solutions too early** - The exploration phase should focus on understanding the domain as it is, not designing the software architecture. Solutions come after understanding.
3. **Treating the result as final** - Event Storming produces a starting point for the domain model, not a finished design. Expect to refine the model as implementation reveals new insights.

## Best Practices

1. **Start with Big Picture Event Storming** - Before diving into a single bounded context, map the entire business flow to understand the landscape and identify context boundaries.
2. **Use physical materials** - Physical sticky notes on a real wall encourage movement, collaboration, and creative rearrangement in ways that digital tools often cannot match.
3. **Capture hotspots** - Mark areas of disagreement or confusion with red stickies. These often reveal the most valuable parts of the domain to model carefully.

## Summary

- Event Storming is a collaborative workshop for exploring complex business domains
- The process moves from chaotic event discovery through commands, aggregates, and policies to bounded contexts
- Orange stickies represent events, blue for commands, yellow for aggregates, and lilac for policies
- Workshop results translate directly into DDD building blocks in code
- Always include domain experts; their knowledge is the most critical input

## Resources

- [Event Storming](https://www.eventstorming.com/) — Alberto Brandolini's official Event Storming website

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*