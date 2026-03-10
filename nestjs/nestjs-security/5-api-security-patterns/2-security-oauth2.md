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

Install Passport.js along with the Google OAuth2 strategy and its TypeScript types.

```bash
npm install @nestjs/passport passport passport-google-oauth20
npm install -D @types/passport-google-oauth20
```

Passport handles the OAuth2 flow complexity — redirects, token exchange, and profile retrieval — so you only need to define what to do with the returned user data.

### Strategy Configuration

Extend `PassportStrategy` with the Google OAuth2 strategy. The `validate()` method is called after a successful authentication and transforms the provider profile into your app's user shape.

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

The second argument `'google'` in `PassportStrategy(Strategy, 'google')` is the strategy name used to reference it in `AuthGuard('google')`.

### Auth Controller

Define two routes: one to initiate the OAuth2 flow and another to handle the callback after the user authorizes your app.

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

The `googleAuth()` method body is empty because the `AuthGuard` handles the redirect to Google. The callback route receives the authorization code and exchanges it for tokens.

### User Linking

Handle three cases: returning OAuth user, existing email user (link accounts), and brand-new user. This prevents duplicate accounts.

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

The email-based account linking step is critical — without it, users who registered with email and later try "Login with Google" would end up with two separate accounts.

### Multiple Providers

Add more OAuth providers by creating additional strategy classes. Each strategy follows the same pattern — only the configuration and profile shape differ.

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

Register all strategy providers in the AuthModule. Passport automatically selects the correct strategy based on the name passed to `AuthGuard('github')` or `AuthGuard('google')`.

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

- OAuth2 delegates authentication to trusted providers (Google, GitHub, etc.) via the Authorization Code flow
- Use Passport.js strategies for each provider and register them in your AuthModule
- Implement account linking to connect OAuth logins with existing email-based accounts
- Always verify email confirmation status from the provider before trusting the address

## Code Examples

**Setting up OAuth2 with Passport.js strategies for Google/GitHub authentication**

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


## Resources

- [Authentication](https://docs.nestjs.com/security/authentication) — Passport and OAuth strategies

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*