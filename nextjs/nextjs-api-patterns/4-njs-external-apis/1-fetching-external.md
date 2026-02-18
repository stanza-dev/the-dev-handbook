---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-fetching-external"
---

# Calling External APIs

Route Handlers are perfect for proxying external API calls.

## Why Proxy?

- **Hide API keys** from the client.
- **Transform data** before sending to the frontend.
- **Combine multiple API calls** into one request.
- **Add caching** for expensive external calls.

## Example: Weather API

```typescript
// app/api/weather/[city]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { city: string } }
) {
  const apiKey = process.env.WEATHER_API_KEY;

  const response = await fetch(
    `https://api.openweathermap.org/data/2.5/weather?q=${params.city}&appid=${apiKey}`,
    { next: { revalidate: 3600 } } // Cache for 1 hour
  );

  if (!response.ok) {
    return Response.json(
      { error: 'City not found' },
      { status: 404 }
    );
  }

  const data = await response.json();

  // Transform data for frontend
  return Response.json({
    city: data.name,
    temp: data.main.temp,
    description: data.weather[0].description,
  });
}
```

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*