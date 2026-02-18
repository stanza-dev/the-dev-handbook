---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-rate-limiting"
---

# Rate Limiting External APIs

## Introduction

External APIs have rate limits. Hit them too fast, and you get blocked. Implementing proper rate limiting and retry logic is essential for reliable integrations.

## Key Concepts

**Rate limiting strategies**:

- **Throttling**: Limit requests per time window
- **Retry with backoff**: Wait longer after each failure
- **Queue**: Process requests sequentially
- **Circuit breaker**: Stop calling failing services temporarily

## Real World Context

Rate limits exist everywhere:
- OpenAI: 60 RPM for GPT-3.5
- GitHub API: 5000 requests/hour
- Stripe: 100 read/s, 100 write/s
- Twitter/X: Various limits

## Deep Dive

### Retry with Exponential Backoff

```typescript
async function fetchWithRetry(
  url: string,
  options: RequestInit = {},
  maxRetries = 3
) {
  let lastError: Error;
  
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      
      if (response.status === 429) {
        // Rate limited - get retry-after header
        const retryAfter = response.headers.get('retry-after');
        const waitTime = retryAfter 
          ? parseInt(retryAfter) * 1000 
          : Math.pow(2, attempt) * 1000;
        
        await new Promise(resolve => setTimeout(resolve, waitTime));
        continue;
      }
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      return response;
    } catch (error) {
      lastError = error as Error;
      
      // Exponential backoff: 1s, 2s, 4s...
      const waitTime = Math.pow(2, attempt) * 1000;
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
  
  throw lastError!;
}
```

### Using in Route Handlers

```typescript
export async function POST(request: Request) {
  const { prompt } = await request.json();
  
  try {
    const response = await fetchWithRetry(
      'https://api.openai.com/v1/chat/completions',
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          model: 'gpt-3.5-turbo',
          messages: [{ role: 'user', content: prompt }],
        }),
      }
    );
    
    const data = await response.json();
    return Response.json(data);
  } catch (error) {
    return Response.json(
      { error: 'AI service temporarily unavailable' },
      { status: 503 }
    );
  }
}
```

### Caching to Reduce Calls

```typescript
import { unstable_cache } from 'next/cache';

const getCachedExchangeRates = unstable_cache(
  async () => {
    const response = await fetch('https://api.exchangerate.host/latest');
    return response.json();
  },
  ['exchange-rates'],
  { revalidate: 3600 } // Cache for 1 hour
);

export async function GET() {
  const rates = await getCachedExchangeRates();
  return Response.json(rates);
}
```

## Common Pitfalls

1. **No retry logic**: A single 429 error shouldn't fail the whole request.

2. **Ignoring Retry-After headers**: APIs tell you when to retry—listen to them.

3. **Aggressive retries**: Rapid retries make rate limiting worse.

## Best Practices

- **Implement exponential backoff**: 1s, 2s, 4s, 8s delays
- **Respect Retry-After headers**: Use them when provided
- **Cache aggressively**: The best request is one you don't make
- **Log rate limit events**: Track when you hit limits

## Summary

External APIs have rate limits that you must respect. Implement retry with exponential backoff, honor Retry-After headers, and cache responses to reduce API calls. A well-designed integration handles rate limits gracefully without failing entire requests.

## Resources

- [Retry Patterns](https://docs.aws.amazon.com/general/latest/gr/api-retries.html) — AWS retry best practices

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*