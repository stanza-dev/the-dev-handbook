---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-webhooks"
---

# Handling Webhooks

## Introduction

Webhooks are how external services notify your application of events—Stripe payments, GitHub pushes, Slack messages. They're incoming POST requests that you need to verify and process.

## Key Concepts

**Webhook security**:

- **Signature verification**: Prove the request came from the claimed sender
- **Idempotency**: Handle duplicate deliveries gracefully
- **Quick response**: Return 200 fast, process async if needed

## Real World Context

Common webhook sources:
- **Stripe**: Payment succeeded, subscription updated
- **GitHub**: Push, PR opened, issue created
- **Clerk**: User created, session started
- **Twilio**: SMS received, call completed

## Deep Dive

### Stripe Webhook Handler

```typescript
// app/webhooks/stripe/route.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(request: Request) {
  const body = await request.text();
  const signature = request.headers.get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
  } catch (error) {
    console.error('Webhook signature verification failed:', error);
    return Response.json({ error: 'Invalid signature' }, { status: 400 });
  }

  // Handle specific event types
  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object as Stripe.Checkout.Session;
      await handleCheckoutComplete(session);
      break;
      
    case 'customer.subscription.updated':
      const subscription = event.data.object as Stripe.Subscription;
      await handleSubscriptionUpdate(subscription);
      break;
      
    default:
      console.log(`Unhandled event type: ${event.type}`);
  }

  return Response.json({ received: true });
}
```

### Idempotent Processing

```typescript
async function handleCheckoutComplete(session: Stripe.Checkout.Session) {
  // Check if already processed
  const existing = await prisma.order.findUnique({
    where: { stripeSessionId: session.id },
  });

  if (existing) {
    console.log(`Order ${session.id} already processed`);
    return;
  }

  // Process the order
  await prisma.order.create({
    data: {
      stripeSessionId: session.id,
      userId: session.metadata?.userId,
      amount: session.amount_total,
      status: 'completed',
    },
  });
}
```

### GitHub Webhook Handler

```typescript
import { createHmac } from 'crypto';

export async function POST(request: Request) {
  const body = await request.text();
  const signature = request.headers.get('x-hub-signature-256')!;
  
  // Verify signature
  const hmac = createHmac('sha256', process.env.GITHUB_WEBHOOK_SECRET!);
  const digest = 'sha256=' + hmac.update(body).digest('hex');
  
  if (signature !== digest) {
    return Response.json({ error: 'Invalid signature' }, { status: 401 });
  }

  const event = request.headers.get('x-github-event');
  const payload = JSON.parse(body);

  switch (event) {
    case 'push':
      await handlePush(payload);
      break;
    case 'pull_request':
      await handlePR(payload);
      break;
  }

  return Response.json({ ok: true });
}
```

### Async Processing

```typescript
export async function POST(request: Request) {
  // Verify and parse webhook...
  
  // Respond immediately
  // Process in background (on serverless, use a queue)
  processWebhookAsync(event).catch(console.error);
  
  return Response.json({ received: true });
}
```

## Common Pitfalls

1. **Not verifying signatures**: Anyone can POST to your webhook URL. Always verify.

2. **Blocking on processing**: Return 200 quickly. Long processing can timeout.

3. **Not handling duplicates**: Webhook providers retry. Same event may arrive multiple times.

## Best Practices

- **Always verify signatures**: Different providers use different methods
- **Store webhook events**: Keep a log for debugging and replay
- **Use idempotency keys**: Process each unique event only once
- **Respond quickly**: Return 200, then process async

## Summary

Webhooks are incoming HTTP requests from external services. Always verify signatures to prevent forgery, handle duplicates with idempotency checks, and return 200 quickly. Store events for debugging and use async processing for long-running tasks.

## Resources

- [Stripe Webhooks](https://stripe.com/docs/webhooks) — Stripe webhook documentation

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*