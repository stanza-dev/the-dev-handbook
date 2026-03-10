---
source_course: "nestjs-security"
source_lesson: "nestjs-security-token-revocation"
---

# Token Revocation

## Introduction

JWTs are stateless—once issued, they're valid until expiration. But what about compromised tokens, password changes, or user-initiated logout? Token revocation adds the ability to invalidate tokens before expiration.

## Key Concepts

- **Token Blacklist**: Store invalidated tokens until they expire
- **Token Version**: Increment version on revocation; reject old versions
- **Short Expiration**: Minimize window of compromised token validity
- **Refresh Token Revocation**: Database-backed refresh token invalidation

## Real World Context

Revocation scenarios:
- User clicks "Logout from all devices"
- Admin disables a compromised account
- Password change invalidates existing sessions
- Suspicious activity detected

## Deep Dive

### Token Versioning Strategy

Embed a `tokenVersion` number in the JWT payload. On each verification, compare it against the user's current version in the database.

```typescript
// User entity
@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ default: 0 })
  tokenVersion: number;
}

// Auth service
@Injectable()
export class AuthService {
  async generateAccessToken(user: User) {
    return this.jwtService.signAsync({
      sub: user.id,
      tokenVersion: user.tokenVersion,
    });
  }

  async verifyAccessToken(token: string) {
    const payload = await this.jwtService.verifyAsync(token);
    const user = await this.usersService.findOne(payload.sub);
    
    if (payload.tokenVersion !== user.tokenVersion) {
      throw new UnauthorizedException('Token has been revoked');
    }
    
    return payload;
  }

  async revokeAllUserTokens(userId: string) {
    await this.usersService.incrementTokenVersion(userId);
  }
}
```

Incrementing the version by one instantly invalidates every existing token for that user without needing to track individual tokens.

### Redis Token Blacklist

For individual token revocation, store blacklisted tokens in Redis with a TTL matching the token's remaining lifetime. Redis provides O(1) lookups.

```typescript
import { Injectable } from '@nestjs/common';
import { Redis } from 'ioredis';

@Injectable()
export class TokenBlacklistService {
  constructor(@InjectRedis() private redis: Redis) {}

  async blacklistToken(token: string, expiresIn: number) {
    // Store token until its natural expiration
    await this.redis.setex(`blacklist:${token}`, expiresIn, '1');
  }

  async isBlacklisted(token: string): Promise<boolean> {
    const result = await this.redis.get(`blacklist:${token}`);
    return result !== null;
  }
}

// In auth guard
@Injectable()
export class JwtAuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    // Check blacklist first (faster than DB lookup)
    if (await this.blacklistService.isBlacklisted(token)) {
      throw new UnauthorizedException('Token has been revoked');
    }
    
    // Then verify JWT
    const payload = await this.authService.verifyAccessToken(token);
    request.user = payload;
    
    return true;
  }
}
```

The `setex` TTL ensures blacklisted tokens are automatically cleaned up when they expire, preventing the blacklist from growing indefinitely.

### Logout All Devices

Combine token versioning with refresh token revocation to immediately invalidate all sessions across every device.

```typescript
@Controller('auth')
export class AuthController {
  @Post('logout-all')
  @UseGuards(JwtAuthGuard)
  async logoutAllDevices(@CurrentUser() user: User) {
    // Increment token version to invalidate all existing tokens
    await this.authService.revokeAllUserTokens(user.id);
    
    // Also revoke all refresh tokens
    await this.refreshTokenService.revokeAllUserTokens(user.id);
    
    return { message: 'Logged out from all devices' };
  }
}
```

Both operations are needed — revoking access tokens prevents immediate API access, while revoking refresh tokens prevents obtaining new access tokens.

### Hybrid Approach (Recommended)

The most practical strategy combines short-lived access tokens (which naturally expire) with database-backed refresh tokens (which are easily revocable).

```typescript
@Injectable()
export class AuthService {
  // Short-lived access tokens (15 min) - rarely need revocation
  // Long-lived refresh tokens - stored in DB, easily revocable
  
  async logout(userId: string, refreshToken: string) {
    // Revoke specific refresh token
    await this.refreshTokenService.revoke(refreshToken);
  }

  async logoutAll(userId: string) {
    // Revoke all refresh tokens
    await this.refreshTokenService.revokeAllUserTokens(userId);
    // Optionally increment token version for immediate access token invalidation
    await this.usersService.incrementTokenVersion(userId);
  }

  async onPasswordChange(userId: string) {
    // Force re-authentication everywhere
    await this.logoutAll(userId);
  }
}
```

With 15-minute access tokens, the maximum window of exposure after a refresh token revocation is 15 minutes — often an acceptable tradeoff to avoid per-request blacklist checks.

## Common Pitfalls

1. **Blacklist grows indefinitely**: Only store until token's original expiration.
2. **Slow blacklist checks**: Use Redis, not database, for blacklist lookups.
3. **Forgetting refresh tokens**: Revoking access tokens without refresh tokens is incomplete.

## Best Practices

- Use short access token expiration (15 min) to minimize revocation need
- Store refresh tokens in database for easy revocation
- Use Redis for high-performance blacklist checks
- Implement "logout all" for security events
- Revoke tokens on password change and suspicious activity

## Summary

- Token versioning enables global revocation by incrementing a version number checked during verification
- Use Redis-backed blacklists for fast, individual token revocation until natural expiration
- Store refresh tokens in the database for easy, granular revocation
- Combine short-lived access tokens with revocable refresh tokens for the best security balance

## Code Examples

**Implementing token blacklisting with version-based invalidation strategy**

```typescript
// User entity
@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ default: 0 })
  tokenVersion: number;
}

// Auth service
@Injectable()
export class AuthService {
  async generateAccessToken(user: User) {
    return this.jwtService.signAsync({
      sub: user.id,
      tokenVersion: user.tokenVersion,
    });
  }

  async verifyAccessToken(token: string) {
    const payload = await this.jwtService.verifyAsync(token);
    const user = await this.usersService.findOne(payload.sub);
    
    if (payload.tokenVersion !== user.tokenVersion) {
      throw new UnauthorizedException('Token has been revoked');
    }
    
    return payload;
  }

  async revokeAllUserTokens(userId: string) {
    await this.usersService.incrementTokenVersion(userId);
  }
}
```


## Resources

- [Authentication](https://docs.nestjs.com/security/authentication) — Token management strategies

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*