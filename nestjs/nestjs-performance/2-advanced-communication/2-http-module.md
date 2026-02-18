---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-http-module"
---

# HTTP Module (Axios)

## Introduction

Most applications need to call external APIs. NestJS wraps Axios in the HttpModule, providing a configured, injectable HttpService that returns Observables and integrates seamlessly with the NestJS ecosystem.

## Key Concepts

- **HttpModule**: Module that provides HttpService
- **HttpService**: Injectable wrapper around Axios
- **Observable**: Returns Observables (convert to Promise if needed)
- **Interceptors**: Axios interceptors for request/response modification

## Deep Dive

### Setup

```bash
npm install @nestjs/axios axios
```

```typescript
import { HttpModule } from '@nestjs/axios';

@Module({
  imports: [HttpModule],
  providers: [ExternalApiService],
})
export class AppModule {}
```

### Basic Usage

```typescript
import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class ExternalApiService {
  constructor(private httpService: HttpService) {}

  // Using Observable
  getDataObservable(): Observable<AxiosResponse<Data>> {
    return this.httpService.get<Data>('https://api.example.com/data');
  }

  // Using Promise
  async getDataPromise(): Promise<Data> {
    const { data } = await firstValueFrom(
      this.httpService.get<Data>('https://api.example.com/data'),
    );
    return data;
  }
}
```

### Configuration

```typescript
HttpModule.register({
  timeout: 5000,
  maxRedirects: 5,
  headers: {
    'User-Agent': 'MyApp/1.0',
  },
})
```

### Async Configuration

```typescript
HttpModule.registerAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    timeout: configService.get('HTTP_TIMEOUT'),
    baseURL: configService.get('API_BASE_URL'),
    headers: {
      'X-API-Key': configService.get('API_KEY'),
    },
  }),
  inject: [ConfigService],
})
```

### Error Handling

```typescript
import { catchError, map } from 'rxjs/operators';
import { AxiosError } from 'axios';

async fetchData(): Promise<Data> {
  try {
    const { data } = await firstValueFrom(
      this.httpService.get<Data>('/endpoint').pipe(
        catchError((error: AxiosError) => {
          this.logger.error('API call failed', error.response?.data);
          throw new HttpException('External API error', HttpStatus.BAD_GATEWAY);
        }),
      ),
    );
    return data;
  } catch (error) {
    throw error;
  }
}
```

### Axios Interceptors

```typescript
@Injectable()
export class ApiService implements OnModuleInit {
  constructor(private httpService: HttpService) {}

  onModuleInit() {
    this.httpService.axiosRef.interceptors.request.use((config) => {
      config.headers['X-Request-ID'] = uuid();
      return config;
    });

    this.httpService.axiosRef.interceptors.response.use(
      (response) => response,
      (error) => {
        this.logger.error('API Error', error.response?.status);
        return Promise.reject(error);
      },
    );
  }
}
```

## Common Pitfalls

1. **Observable vs Promise**: HttpService returns Observables. Use firstValueFrom() for Promises.
2. **Unsubscribed Observables**: Observables not subscribed don't execute.
3. **Missing error handling**: Network errors can crash your app without try/catch.

## Best Practices

- Use firstValueFrom() for simple request/response patterns
- Configure timeouts to prevent hanging requests
- Add logging interceptors for debugging
- Handle errors gracefully with meaningful messages

## Summary

HttpModule provides HttpService for making HTTP requests. It returns Observables—use firstValueFrom() for Promise-style code. Configure timeouts, base URLs, and headers. Handle errors gracefully and use interceptors for cross-cutting concerns.

## Resources

- [HTTP Module](https://docs.nestjs.com/techniques/http-module) — Official HTTP module guide

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*