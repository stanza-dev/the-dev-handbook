---
source_course: "nestjs-security"
source_lesson: "nestjs-security-authentication"
---

# Authentication (JWT)

## Introduction

Authentication answers the question "Who are you?" It's the foundation of API security. Without proper authentication, anyone can access your protected resources. JWT (JSON Web Token) is the industry standard for stateless authentication in modern APIs.

## Key Concepts

- **Authentication**: Verifying the identity of a user or system
- **JWT (JSON Web Token)**: A compact, URL-safe token format for transmitting claims
- **Access Token**: Short-lived token for API access (typically 15 min to 1 hour)
- **Refresh Token**: Long-lived token for obtaining new access tokens
- **Guards**: NestJS mechanism for protecting routes

## Real World Context

Every production API needs authentication:
- Mobile apps authenticating users
- SPAs calling backend APIs
- Service-to-service communication in microservices
- Third-party integrations

JWT's stateless nature makes it ideal for distributed systems where session storage is impractical.

## Deep Dive

### Required Packages

```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install -D @types/passport-jwt
```

### Auth Service Implementation

```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async signIn(email: string, password: string) {
    const user = await this.usersService.findByEmail(email);
    
    if (!user || !(await bcrypt.compare(password, user.password))) {
      throw new UnauthorizedException('Invalid credentials');
    }
    
    const payload = { sub: user.id, email: user.email };
    
    return {
      access_token: await this.jwtService.signAsync(payload),
    };
  }
}
```

### JWT Auth Guard

```typescript
import { Injectable, CanActivate, ExecutionContext, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { Request } from 'express';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);
    
    if (!token) {
      throw new UnauthorizedException();
    }
    
    try {
      const payload = await this.jwtService.verifyAsync(token);
      request['user'] = payload;
    } catch {
      throw new UnauthorizedException();
    }
    
    return true;
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}
```

### Protecting Routes

```typescript
@Controller('users')
export class UsersController {
  @UseGuards(AuthGuard)
  @Get('profile')
  getProfile(@Request() req) {
    return req.user;
  }
}
```

### JWT Module Configuration

```typescript
@Module({
  imports: [
    JwtModule.register({
      global: true,
      secret: process.env.JWT_SECRET,
      signOptions: { expiresIn: '15m' },
    }),
  ],
})
export class AuthModule {}
```

## Common Pitfalls

1. **Hardcoding secrets**: Never commit JWT secrets to source control. Use environment variables.
2. **Long-lived access tokens**: Access tokens should expire quickly (15-60 min). Use refresh tokens for longer sessions.
3. **Not validating token structure**: Always verify the token signature and expiration.

## Best Practices

- Store JWT secrets in environment variables or secret managers
- Use short expiration times for access tokens
- Implement refresh token rotation for security
- Hash passwords with bcrypt (cost factor 10+)
- Return generic error messages to prevent user enumeration

## Summary

JWT authentication in NestJS involves creating an AuthService to validate credentials and issue tokens, and an AuthGuard to protect routes. The @nestjs/jwt package handles token signing and verification. Always use environment variables for secrets and implement proper token expiration.

## Resources

- [Authentication Documentation](https://docs.nestjs.com/security/authentication) — Official guide to authentication
- [JWT Package](https://docs.nestjs.com/security/authentication#jwt-token) — JWT implementation details

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*