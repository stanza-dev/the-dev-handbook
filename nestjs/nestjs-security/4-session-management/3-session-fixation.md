---
source_course: "nestjs-security"
source_lesson: "nestjs-security-session-fixation"
---

# Session Fixation Prevention

## Introduction

Session fixation attacks trick users into using attacker-controlled session IDs. When the victim logs in, the attacker gains access using the same session. Prevention requires regenerating sessions on authentication.

## Key Concepts

- **Session Fixation**: Attack where attacker sets victim's session ID
- **Session Regeneration**: Creating new session ID on privilege change
- **Session Binding**: Tying sessions to device/IP characteristics
- **Absolute Timeout**: Maximum session lifetime regardless of activity

## Real World Context

Session fixation scenarios:
- Attacker sends link with session ID in URL
- Attacker sets cookie via XSS before victim logs in
- Shared computers with leftover sessions

## Deep Dive

### Session Regeneration on Login

```typescript
@Injectable()
export class AuthService {
  async login(request: Request, email: string, password: string) {
    const user = await this.validateCredentials(email, password);
    
    // Regenerate session on successful login
    await this.regenerateSession(request);
    
    // Store user in new session
    request.session.userId = user.id;
    request.session.loginTime = Date.now();
    request.session.userAgent = request.headers['user-agent'];
    request.session.ip = request.ip;
    
    return user;
  }

  private regenerateSession(request: Request): Promise<void> {
    return new Promise((resolve, reject) => {
      const oldSession = { ...request.session };
      request.session.regenerate((err) => {
        if (err) reject(err);
        // Optionally copy non-sensitive data
        resolve();
      });
    });
  }
}
```

### Session Binding Verification

```typescript
@Injectable()
export class SessionGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const session = request.session;
    
    if (!session?.userId) {
      throw new UnauthorizedException('No active session');
    }
    
    // Verify session binding
    if (session.ip !== request.ip) {
      // Log suspicious activity
      this.logger.warn(`Session IP mismatch: ${session.ip} vs ${request.ip}`);
      // Optionally invalidate or require re-auth
    }
    
    // Check absolute timeout (24 hours)
    const sessionAge = Date.now() - session.loginTime;
    if (sessionAge > 24 * 60 * 60 * 1000) {
      request.session.destroy();
      throw new UnauthorizedException('Session expired');
    }
    
    return true;
  }
}
```

### JWT-Based Session Equivalent

For stateless JWT auth, regenerate tokens on privilege changes:

```typescript
@Injectable()
export class AuthService {
  async changePassword(userId: string, newPassword: string) {
    await this.usersService.updatePassword(userId, newPassword);
    
    // Increment token version to invalidate old tokens
    await this.usersService.incrementTokenVersion(userId);
    
    // Generate new tokens
    return this.generateTokens(userId);
  }

  async verifyAccessToken(token: string) {
    const payload = await this.jwtService.verifyAsync(token);
    
    // Check token version matches current user version
    const user = await this.usersService.findOne(payload.sub);
    if (payload.tokenVersion !== user.tokenVersion) {
      throw new UnauthorizedException('Token invalidated');
    }
    
    return payload;
  }
}
```

### Absolute Session Timeout

```typescript
@Injectable()
export class SessionTimeoutInterceptor implements NestInterceptor {
  private readonly ABSOLUTE_TIMEOUT = 24 * 60 * 60 * 1000; // 24 hours
  private readonly IDLE_TIMEOUT = 30 * 60 * 1000; // 30 minutes

  intercept(context: ExecutionContext, next: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const session = request.session;
    
    if (session?.userId) {
      const now = Date.now();
      
      // Absolute timeout
      if (now - session.loginTime > this.ABSOLUTE_TIMEOUT) {
        session.destroy();
        throw new UnauthorizedException('Session expired');
      }
      
      // Idle timeout
      if (now - session.lastActivity > this.IDLE_TIMEOUT) {
        session.destroy();
        throw new UnauthorizedException('Session timed out due to inactivity');
      }
      
      // Update last activity
      session.lastActivity = now;
    }
    
    return next.handle();
  }
}
```

## Common Pitfalls

1. **Not regenerating on login**: The core vulnerability. Always regenerate session ID.
2. **IP binding too strict**: Mobile users change IPs. Be flexible or use other bindings.
3. **No absolute timeout**: Even active sessions should eventually expire.

## Best Practices

- Regenerate session ID on every authentication event
- Regenerate on privilege escalation (admin login, password change)
- Implement both idle and absolute timeouts
- Bind sessions to user-agent for additional security
- Log session anomalies for security monitoring

## Summary

Prevent session fixation by regenerating session IDs on login and privilege changes. Implement absolute timeouts alongside idle timeouts. For JWT, use token versioning to invalidate old tokens. Log and monitor session anomalies.

## Resources

- [Security Best Practices](https://docs.nestjs.com/security/authentication) — Session security patterns

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*