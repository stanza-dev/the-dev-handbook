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

### API Key Guard

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

### Scope-Based Authorization

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

### Key Rotation

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

API keys authenticate applications, not users. Generate cryptographically secure keys, store them hashed, and implement scope-based authorization. Support key rotation for security and use headers (not URLs) for key transmission.

## Resources

- [Authentication](https://docs.nestjs.com/security/authentication) — Custom authentication strategies

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*