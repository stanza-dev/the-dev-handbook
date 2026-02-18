---
source_course: "nestjs-security"
source_lesson: "nestjs-security-cookie-security"
---

# Cookie Security

## Introduction

Cookies carry sensitive data like session IDs and tokens. Improper cookie configuration exposes users to session hijacking, XSS token theft, and CSRF attacks. Secure cookies are foundational to web security.

## Key Concepts

- **HttpOnly**: Cookie inaccessible to JavaScript (prevents XSS theft)
- **Secure**: Cookie only sent over HTTPS
- **SameSite**: Controls cross-site cookie sending (CSRF protection)
- **Domain/Path**: Scope of cookie visibility

## Real World Context

Cookie security prevents:
- Session hijacking via network sniffing (Secure flag)
- Token theft via XSS (HttpOnly flag)
- CSRF attacks (SameSite flag)

## Deep Dive

### Secure Cookie Configuration

```typescript
import { Response } from 'express';

@Injectable()
export class AuthService {
  setRefreshTokenCookie(response: Response, token: string) {
    response.cookie('refreshToken', token, {
      httpOnly: true,        // No JavaScript access
      secure: process.env.NODE_ENV === 'production', // HTTPS only
      sameSite: 'strict',    // CSRF protection
      maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
      path: '/auth/refresh', // Only sent to refresh endpoint
    });
  }

  clearRefreshTokenCookie(response: Response) {
    response.clearCookie('refreshToken', {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      path: '/auth/refresh',
    });
  }
}
```

### Cookie Parser Setup

```typescript
import * as cookieParser from 'cookie-parser';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(cookieParser(process.env.COOKIE_SECRET));
  await app.listen(3000);
}
```

### Signed Cookies

```typescript
// Setting a signed cookie
response.cookie('userId', user.id, {
  signed: true,
  httpOnly: true,
  secure: true,
});

// Reading signed cookie
@Get('profile')
profile(@Req() request: Request) {
  const userId = request.signedCookies.userId;
  // If tampered, value will be false
  if (!userId) throw new UnauthorizedException();
  return this.usersService.findOne(userId);
}
```

### SameSite Options Explained

| Value | Behavior | Use Case |
|-------|----------|----------|
| `strict` | Never sent cross-site | Session cookies, maximum security |
| `lax` | Sent with top-level navigation | Default, balanced security/usability |
| `none` | Always sent (requires Secure) | Cross-site APIs, third-party widgets |

### Cookie-Based Authentication Flow

```typescript
@Controller('auth')
export class AuthController {
  @Post('login')
  async login(
    @Body() dto: LoginDto,
    @Res({ passthrough: true }) response: Response,
  ) {
    const tokens = await this.authService.login(dto.email, dto.password);
    
    // Store refresh token in httpOnly cookie
    this.authService.setRefreshTokenCookie(response, tokens.refreshToken);
    
    // Return access token in body (for Authorization header)
    return { accessToken: tokens.accessToken };
  }

  @Post('refresh')
  async refresh(
    @Req() request: Request,
    @Res({ passthrough: true }) response: Response,
  ) {
    const refreshToken = request.cookies.refreshToken;
    if (!refreshToken) throw new UnauthorizedException();
    
    const tokens = await this.authService.refreshTokens(refreshToken);
    this.authService.setRefreshTokenCookie(response, tokens.refreshToken);
    
    return { accessToken: tokens.accessToken };
  }

  @Post('logout')
  logout(@Res({ passthrough: true }) response: Response) {
    this.authService.clearRefreshTokenCookie(response);
    return { message: 'Logged out' };
  }
}
```

## Common Pitfalls

1. **Missing HttpOnly**: Tokens accessible to JavaScript can be stolen via XSS.
2. **SameSite=None without Secure**: Browsers reject this combination.
3. **Wrong path**: Cookies scoped incorrectly may not be sent or be sent too broadly.

## Best Practices

- Always use HttpOnly for session/auth cookies
- Always use Secure in production
- Use SameSite=Strict for session cookies
- Scope cookies to specific paths when possible
- Use signed cookies for integrity verification
- Clear cookies on logout with same options

## Summary

Secure cookies use HttpOnly (no JS access), Secure (HTTPS only), and SameSite (CSRF protection). Store refresh tokens in httpOnly cookies, access tokens in memory. Always clear cookies with the same options used to set them.

## Resources

- [Security](https://docs.nestjs.com/security/authentication) — Cookie and session patterns

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*