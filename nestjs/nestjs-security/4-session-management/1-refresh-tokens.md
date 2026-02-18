---
source_course: "nestjs-security"
source_lesson: "nestjs-security-refresh-tokens"
---

# Refresh Token Strategy

## Introduction

Short-lived access tokens are secure but inconvenient—users hate logging in repeatedly. Refresh tokens provide the balance: short-lived access tokens (minutes) with long-lived refresh tokens (days/weeks) for seamless re-authentication.

## Key Concepts

- **Access Token**: Short-lived JWT for API access (15 min - 1 hour)
- **Refresh Token**: Long-lived token for obtaining new access tokens
- **Token Rotation**: Issue new refresh token on each use
- **Token Family**: Track refresh token lineage for breach detection

## Real World Context

Refresh tokens power:
- Mobile apps staying logged in for weeks
- Web SPAs maintaining sessions across browser restarts
- Detecting token theft through rotation

## Deep Dive

### Token Pair Generation

```typescript
@Injectable()
export class AuthService {
  constructor(
    private jwtService: JwtService,
    private configService: ConfigService,
  ) {}

  async generateTokens(userId: string) {
    const [accessToken, refreshToken] = await Promise.all([
      this.jwtService.signAsync(
        { sub: userId, type: 'access' },
        {
          secret: this.configService.get('JWT_ACCESS_SECRET'),
          expiresIn: '15m',
        },
      ),
      this.jwtService.signAsync(
        { sub: userId, type: 'refresh' },
        {
          secret: this.configService.get('JWT_REFRESH_SECRET'),
          expiresIn: '7d',
        },
      ),
    ]);

    return { accessToken, refreshToken };
  }
}
```

### Refresh Token Storage

```typescript
@Injectable()
export class RefreshTokenService {
  constructor(
    @InjectRepository(RefreshToken)
    private refreshTokenRepo: Repository<RefreshToken>,
  ) {}

  async storeRefreshToken(userId: string, token: string, family: string) {
    const hashedToken = await bcrypt.hash(token, 10);
    const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000); // 7 days

    await this.refreshTokenRepo.save({
      userId,
      hashedToken,
      family,
      expiresAt,
    });
  }

  async validateAndRotate(userId: string, token: string, family: string) {
    const storedToken = await this.refreshTokenRepo.findOne({
      where: { userId, family },
      order: { createdAt: 'DESC' },
    });

    if (!storedToken || storedToken.revoked) {
      // Possible token theft - revoke entire family
      await this.revokeFamily(family);
      throw new UnauthorizedException('Invalid refresh token');
    }

    const isValid = await bcrypt.compare(token, storedToken.hashedToken);
    if (!isValid) {
      await this.revokeFamily(family);
      throw new UnauthorizedException('Invalid refresh token');
    }

    // Revoke used token
    await this.refreshTokenRepo.update(storedToken.id, { revoked: true });

    return true;
  }

  async revokeFamily(family: string) {
    await this.refreshTokenRepo.update({ family }, { revoked: true });
  }
}
```

### Refresh Endpoint

```typescript
@Controller('auth')
export class AuthController {
  @Post('refresh')
  async refreshTokens(
    @Body('refreshToken') refreshToken: string,
    @Res({ passthrough: true }) response: Response,
  ) {
    const payload = await this.authService.verifyRefreshToken(refreshToken);
    
    await this.refreshTokenService.validateAndRotate(
      payload.sub,
      refreshToken,
      payload.family,
    );

    const tokens = await this.authService.generateTokens(payload.sub);
    
    await this.refreshTokenService.storeRefreshToken(
      payload.sub,
      tokens.refreshToken,
      payload.family,
    );

    return tokens;
  }
}
```

### Token Family for Breach Detection

```typescript
async login(email: string, password: string) {
  const user = await this.validateUser(email, password);
  const family = randomUUID(); // New family on login
  
  const tokens = await this.generateTokens(user.id);
  
  await this.refreshTokenService.storeRefreshToken(
    user.id,
    tokens.refreshToken,
    family,
  );

  return tokens;
}
```

## Common Pitfalls

1. **Storing refresh tokens in JWT only**: Store in database to enable revocation.
2. **No rotation**: Without rotation, stolen tokens work forever.
3. **Same secret for both tokens**: Use different secrets for access and refresh tokens.

## Best Practices

- Use different secrets for access vs refresh tokens
- Implement token rotation on every refresh
- Store refresh tokens hashed in database
- Use token families to detect and revoke on breach
- Set appropriate expiration times

## Summary

Refresh tokens enable long-lived sessions with short-lived access tokens. Store refresh tokens hashed in database, rotate on every use, and use token families for breach detection. Revoke entire families when reuse is detected.

## Resources

- [Authentication](https://docs.nestjs.com/security/authentication) — JWT authentication patterns

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*