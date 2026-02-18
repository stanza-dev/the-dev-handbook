---
source_course: "nestjs-security"
source_lesson: "nestjs-security-oauth2"
---

# OAuth2 Integration

## Introduction

OAuth2 enables users to authenticate via third-party providers (Google, GitHub, Facebook) without sharing passwords. Users trust familiar login screens, and you don't store sensitive credentials. Passport.js makes OAuth2 integration straightforward in NestJS.

## Key Concepts

- **OAuth2**: Authorization framework for delegated access
- **Authorization Code Flow**: Secure flow for server-side apps
- **Access Token**: Token from provider for accessing user data
- **Passport Strategy**: Passport.js plugin for specific providers

## Real World Context

OAuth2 powers:
- "Login with Google/GitHub/Facebook" buttons
- Third-party app integrations
- Single Sign-On (SSO) systems

## Deep Dive

### Google OAuth2 Setup

```bash
npm install @nestjs/passport passport passport-google-oauth20
npm install -D @types/passport-google-oauth20
```

### Strategy Configuration

```typescript
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { Strategy, VerifyCallback } from 'passport-google-oauth20';

@Injectable()
export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
  constructor(private configService: ConfigService) {
    super({
      clientID: configService.get('GOOGLE_CLIENT_ID'),
      clientSecret: configService.get('GOOGLE_CLIENT_SECRET'),
      callbackURL: configService.get('GOOGLE_CALLBACK_URL'),
      scope: ['email', 'profile'],
    });
  }

  async validate(
    accessToken: string,
    refreshToken: string,
    profile: any,
    done: VerifyCallback,
  ): Promise<any> {
    const { name, emails, photos } = profile;
    
    const user = {
      email: emails[0].value,
      firstName: name.givenName,
      lastName: name.familyName,
      picture: photos[0].value,
      accessToken,
    };
    
    done(null, user);
  }
}
```

### Auth Controller

```typescript
import { AuthGuard } from '@nestjs/passport';

@Controller('auth')
export class AuthController {
  @Get('google')
  @UseGuards(AuthGuard('google'))
  googleAuth() {
    // Initiates Google OAuth2 flow
  }

  @Get('google/callback')
  @UseGuards(AuthGuard('google'))
  async googleAuthCallback(@Req() req, @Res() res: Response) {
    // req.user contains the validated user from strategy
    const user = await this.authService.findOrCreateOAuthUser(req.user);
    const tokens = await this.authService.generateTokens(user.id);
    
    // Redirect to frontend with tokens
    res.redirect(
      `${this.configService.get('FRONTEND_URL')}/auth/callback?token=${tokens.accessToken}`,
    );
  }
}
```

### User Linking

```typescript
@Injectable()
export class AuthService {
  async findOrCreateOAuthUser(profile: OAuthProfile) {
    // Check if user exists by OAuth provider ID
    let user = await this.usersService.findByOAuthId(
      profile.provider,
      profile.providerId,
    );
    
    if (user) {
      return user;
    }
    
    // Check if user exists by email (link accounts)
    user = await this.usersService.findByEmail(profile.email);
    
    if (user) {
      // Link OAuth provider to existing account
      await this.usersService.linkOAuthProvider(user.id, {
        provider: profile.provider,
        providerId: profile.providerId,
      });
      return user;
    }
    
    // Create new user
    return this.usersService.createFromOAuth(profile);
  }
}
```

### Multiple Providers

```typescript
// GitHub Strategy
@Injectable()
export class GitHubStrategy extends PassportStrategy(Strategy, 'github') {
  constructor(configService: ConfigService) {
    super({
      clientID: configService.get('GITHUB_CLIENT_ID'),
      clientSecret: configService.get('GITHUB_CLIENT_SECRET'),
      callbackURL: configService.get('GITHUB_CALLBACK_URL'),
      scope: ['user:email'],
    });
  }

  async validate(accessToken: string, refreshToken: string, profile: any) {
    return {
      provider: 'github',
      providerId: profile.id,
      email: profile.emails[0].value,
      name: profile.displayName,
    };
  }
}

// Module registration
@Module({
  providers: [GoogleStrategy, GitHubStrategy],
})
export class AuthModule {}
```

## Common Pitfalls

1. **Trusting email without verification**: Providers may return unverified emails.
2. **Not handling account linking**: Users may sign up with email, then try OAuth.
3. **Exposing tokens in URLs**: Use short-lived codes or POST to exchange tokens.

## Best Practices

- Verify email is confirmed from OAuth provider
- Implement account linking for existing users
- Store provider access tokens securely if needed for API calls
- Use state parameter for CSRF protection
- Support multiple OAuth providers

## Summary

OAuth2 delegates authentication to trusted providers. Use Passport strategies for each provider, implement account linking for existing users, and handle the callback to exchange tokens. Always verify email confirmation status.

## Resources

- [Authentication](https://docs.nestjs.com/security/authentication) — Passport and OAuth strategies

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*