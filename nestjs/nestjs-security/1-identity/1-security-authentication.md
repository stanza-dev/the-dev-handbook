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

Install the official NestJS JWT package to handle token signing and verification.

```bash
npm install @nestjs/jwt
```

This single package provides everything you need for basic JWT authentication.

> For more complex authentication flows with OAuth providers, you can also use `@nestjs/passport` and provider-specific strategies.

### Auth Service Implementation

The AuthService handles credential validation and token issuance. It uses bcrypt to compare the submitted password against the stored hash.

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

Notice the generic error message "Invalid credentials" — never reveal whether the email or password was wrong, as that helps attackers enumerate valid accounts.

### JWT Auth Guard

The guard extracts the Bearer token from the Authorization header and verifies it. If valid, it attaches the decoded payload to the request object.

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

The key line is `request['user'] = payload` — this makes the decoded user data available to all downstream handlers and guards.

### Protecting Routes

Apply the guard to any route using the `@UseGuards()` decorator. The authenticated user data is then available via `req.user`.

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

You can apply `@UseGuards()` at the method level (as shown) or at the controller level to protect all routes in the controller.

### JWT Module Configuration

Register the JwtModule globally so the JwtService is available throughout the application without re-importing.

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

Setting `global: true` eliminates the need to import JwtModule in every module that uses the JwtService. The `expiresIn: '15m'` sets a sensible default expiration.

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

- Create an AuthService that validates credentials and issues JWTs using `@nestjs/jwt`
- Implement an AuthGuard that extracts Bearer tokens from the Authorization header and verifies them
- Store JWT secrets in environment variables and use short expiration times (15-60 min)
- Hash passwords with bcrypt and return generic error messages to prevent user enumeration

## Code Examples

**Implementing JWT authentication with signIn method and token verification guard**

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


## Resources

- [Authentication Documentation](https://docs.nestjs.com/security/authentication) — Official guide to authentication
- [JWT Package](https://docs.nestjs.com/security/authentication#jwt-token) — JWT implementation details

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*