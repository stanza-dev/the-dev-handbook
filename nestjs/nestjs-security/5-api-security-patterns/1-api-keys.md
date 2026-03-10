---
source_course: "nestjs-security"
source_lesson: "nestjs-security-api-keys"
---

# API Key Authentication

## Introduction

API keys are simple authentication tokens for machine-to-machine communication. Unlike user JWTs, API keys identify applications or services. They're ideal for rate limiting, usage tracking, and granting programmatic access to your API.

## Key Concepts

- **API Key**: Long-lived token identifying an application
- **Key Hashing**: Store hashed keys, not plaintext
- **Key Scopes**: Permissions associated with keys
- **Key Rotation**: Replacing keys without downtime

## Real World Context

API keys power:
- Third-party integrations
- Service-to-service authentication
- Developer portal access
- Webhook authentication

## Deep Dive

### API Key Generation

Generate API keys with a recognizable prefix (`sk_live_`) followed by cryptographically random bytes. Hash the key before storing it.

```typescript
import * as crypto from 'crypto';

@Injectable()
export class ApiKeyService {
  async generateApiKey(): Promise<{ key: string; hashedKey: string }> {
    const key = `sk_live_${crypto.randomBytes(32).toString('hex')}`;
    const hashedKey = crypto
      .createHash('sha256')
      .update(key)
      .digest('hex');
    
    return { key, hashedKey };
  }

  async createApiKey(userId: string, name: string, scopes: string[]) {
    const { key, hashedKey } = await this.generateApiKey();
    
    await this.apiKeyRepo.save({
      userId,
      name,
      hashedKey,
      scopes,
      prefix: key.substring(0, 12), // Store prefix for identification
    });
    
    // Return full key ONLY ONCE - user must save it
    return { key, prefix: key.substring(0, 12) };
  }
}
```

The full key is returned only once at creation time. Storing a short prefix allows users to identify which key is which without exposing the secret.

### API Key Guard

The guard checks for the API key in the `X-API-Key` header first, then falls back to the `Authorization: Bearer` header for flexibility.

```typescript
@Injectable()
export class ApiKeyGuard implements CanActivate {
  constructor(private apiKeyService: ApiKeyService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const apiKey = this.extractApiKey(request);
    
    if (!apiKey) {
      throw new UnauthorizedException('API key required');
    }
    
    const keyData = await this.apiKeyService.validateKey(apiKey);
    if (!keyData) {
      throw new UnauthorizedException('Invalid API key');
    }
    
    // Attach key data to request
    request.apiKey = keyData;
    return true;
  }

  private extractApiKey(request: Request): string | undefined {
    // Check header first
    const headerKey = request.headers['x-api-key'];
    if (headerKey) return headerKey as string;
    
    // Check Authorization header
    const authHeader = request.headers.authorization;
    if (authHeader?.startsWith('Bearer sk_')) {
      return authHeader.substring(7);
    }
    
    return undefined;
  }
}
```

Attaching `keyData` to the request allows downstream guards and handlers to access the key's scopes, owner, and metadata.

### Scope-Based Authorization

Implement fine-grained permissions using scopes. The `@RequiredScopes()` decorator specifies which scopes an endpoint needs, and the guard enforces them.

```typescript
export const RequiredScopes = (...scopes: string[]) =>
  SetMetadata('requiredScopes', scopes);

@Injectable()
export class ApiScopesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredScopes = this.reflector.get<string[]>(
      'requiredScopes',
      context.getHandler(),
    );
    
    if (!requiredScopes) return true;
    
    const request = context.switchToHttp().getRequest();
    const keyScopes = request.apiKey?.scopes || [];
    
    const hasScopes = requiredScopes.every(scope =>
      keyScopes.includes(scope),
    );
    
    if (!hasScopes) {
      throw new ForbiddenException('Insufficient API key scopes');
    }
    
    return true;
  }
}

// Usage
@Get('users')
@UseGuards(ApiKeyGuard, ApiScopesGuard)
@RequiredScopes('users:read')
findAllUsers() { ... }
```

The guard uses `every()` to require all listed scopes — a key must have `users:read` AND any other required scopes, not just one of them.

### Key Rotation

Support zero-downtime key rotation by creating a new key and marking the old one as deprecated with a grace period.

```typescript
@Injectable()
export class ApiKeyService {
  async rotateKey(keyId: string) {
    const existingKey = await this.apiKeyRepo.findOne(keyId);
    if (!existingKey) throw new NotFoundException();
    
    const { key, hashedKey } = await this.generateApiKey();
    
    // Create new key
    const newKey = await this.apiKeyRepo.save({
      ...existingKey,
      id: undefined,
      hashedKey,
      prefix: key.substring(0, 12),
      previousKeyId: keyId,
    });
    
    // Mark old key for deprecation (still valid for grace period)
    await this.apiKeyRepo.update(keyId, {
      deprecatedAt: new Date(),
      replacedByKeyId: newKey.id,
    });
    
    return { key, prefix: key.substring(0, 12) };
  }
}
```

The `previousKeyId` and `replacedByKeyId` fields create a chain of key versions, making it easy to track rotation history during audits.

## Common Pitfalls

1. **Storing keys in plaintext**: Always hash API keys.
2. **No key rotation**: Keys should be rotatable without downtime.
3. **Keys in URLs**: Keys in URLs appear in logs. Use headers.

## Best Practices

- Hash API keys with SHA-256 before storage
- Store a prefix for key identification without revealing the key
- Implement scopes for fine-grained permissions
- Support key rotation with grace periods
- Log API key usage for auditing
- Rate limit by API key

## Summary

- API keys authenticate applications (not users) for machine-to-machine communication
- Generate keys with `crypto.randomBytes`, hash with SHA-256 before storage, and show the full key only once
- Implement scope-based authorization to grant fine-grained permissions per key
- Support key rotation with grace periods and always transmit keys via headers, never URLs

## Code Examples

**Implementing API key authentication with a custom guard and hashed storage**

```typescript
import * as crypto from 'crypto';

@Injectable()
export class ApiKeyService {
  async generateApiKey(): Promise<{ key: string; hashedKey: string }> {
    const key = `sk_live_${crypto.randomBytes(32).toString('hex')}`;
    const hashedKey = crypto
      .createHash('sha256')
      .update(key)
      .digest('hex');
    
    return { key, hashedKey };
  }

  async createApiKey(userId: string, name: string, scopes: string[]) {
    const { key, hashedKey } = await this.generateApiKey();
    
    await this.apiKeyRepo.save({
      userId,
      name,
      hashedKey,
      scopes,
      prefix: key.substring(0, 12), // Store prefix for identification
    });
    
    // Return full key ONLY ONCE - user must save it
    return { key, prefix: key.substring(0, 12) };
  }
}
```


## Resources

- [Authentication](https://docs.nestjs.com/security/authentication) — Custom authentication strategies

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*