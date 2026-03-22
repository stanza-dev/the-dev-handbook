---
source_course: "postgresql-security"
source_lesson: "postgresql-security-oauth-auth"
---

# OAuth Authentication

## Introduction
PostgreSQL 18 introduces OAuth as a first-class authentication method, allowing users to authenticate against external identity providers (IdPs) like Okta, Azure AD, or Keycloak. This is a major step forward for enterprises that want to unify database access under their existing single sign-on (SSO) infrastructure.

## Key Concepts
- **OAuth 2.0**: An authorization framework that lets a third-party application access resources on behalf of a user without sharing credentials.
- **OpenID Connect (OIDC)**: A layer on top of OAuth 2.0 that provides identity verification — used by PostgreSQL 18 for token validation.
- **Bearer Token**: The JWT or opaque token that the client obtains from the identity provider and presents to PostgreSQL for authentication.

## Real World Context
Before PostgreSQL 18, integrating database access with corporate SSO required workarounds like LDAP proxying or custom PAM modules. With native OAuth support, a developer can authenticate to PostgreSQL using the same credentials they use for their company's web applications. This simplifies onboarding, offboarding, and audit trails because user lifecycle is managed centrally in the identity provider.

## Deep Dive

### Configuring OAuth in pg_hba.conf

The `oauth` method in pg_hba.conf tells PostgreSQL to accept bearer tokens validated against a configured issuer:

```conf
# pg_hba.conf - enable OAuth for remote connections
hostssl  all  all  0.0.0.0/0  oauth
```

### Server Configuration

In `postgresql.conf`, configure the OAuth validator library:

```ini
# postgresql.conf
oauth_validator_libraries = 'your_validator_module'
```

PostgreSQL 18 provides the `oauth_validator_libraries` parameter and a framework for building validator modules (see Chapter 50 of the docs), but does not ship a built-in validator. You must install a third-party or custom validator module. The exact configuration depends on your identity provider and the validator module you choose.

### Client Connection Flow

The OAuth flow for PostgreSQL works as follows:

1. The client application obtains an access token from the identity provider (e.g., via OAuth 2.0 device authorization or authorization code flow).
2. The client presents the bearer token to PostgreSQL during the SASL authentication exchange.
3. PostgreSQL's validator module verifies the token signature, expiration, issuer, and audience claims.
4. If valid, the connection is established with the role mapped from the token.

```bash
# Example connection with libpq (conceptual)
# The client library handles token acquisition
psql "host=db.example.com dbname=myapp oauth_issuer=https://idp.example.com oauth_client_id=pg-app"
```

### Mapping Tokens to Roles

PostgreSQL maps the authenticated identity from the token to a database role. You can configure this mapping using `pg_ident.conf` or let the token's subject claim match the role name directly.

```conf
# pg_ident.conf - map OAuth subjects to database roles
# MAPNAME    SYSTEM-USERNAME       PG-USERNAME
oauth_map    alice@example.com     alice
oauth_map    bob@example.com       bob_readonly
```

### Security Considerations

OAuth tokens have expiration times, which provides built-in credential rotation. However, be aware:
- Always use `hostssl` with OAuth to protect bearer tokens in transit.
- Validate the `aud` (audience) claim to prevent token reuse from other applications.
- Token revocation requires checking with the IdP, which adds latency.

## Common Pitfalls
1. **Using OAuth over unencrypted connections** — Bearer tokens in plaintext can be intercepted. Always require SSL with `hostssl` when using OAuth.
2. **Not validating the audience claim** — Without audience validation, a token issued for a different application could be used to access your database.

## Best Practices
1. **Combine OAuth with hostssl** — Ensure tokens are always transmitted over encrypted connections.
2. **Use short-lived tokens with refresh** — Configure your IdP to issue tokens with short expiration (5-15 minutes) and use refresh tokens to minimize exposure if a token is compromised.

## Summary
- PostgreSQL 18 adds native OAuth authentication, enabling integration with identity providers like Okta, Azure AD, and Keycloak.
- The client obtains a bearer token from the IdP and presents it during connection, eliminating the need to store database passwords.
- Always use SSL, validate audience claims, and prefer short-lived tokens for maximum security.

## Code Examples

**Configuration entries for enabling OAuth authentication in PostgreSQL 18**

```sql
-- pg_hba.conf entry for OAuth
-- hostssl  all  all  0.0.0.0/0  oauth

-- postgresql.conf setting (validator module must be installed separately)
-- oauth_validator_libraries = 'your_validator_module'

-- Check current auth method for a connection
SELECT auth_method FROM pg_hba_file_rules
WHERE auth_method = 'oauth';
```

**Mapping OAuth token subjects to PostgreSQL roles via pg_ident.conf**

```sql
-- pg_ident.conf mapping for OAuth subjects to roles
-- MAPNAME    SYSTEM-USERNAME       PG-USERNAME
-- oauth_map  alice@example.com     alice
-- oauth_map  bob@example.com       bob_readonly

-- Verify identity mappings
SELECT * FROM pg_ident_file_mappings;
```


## Resources

- [OAuth Authentication](https://www.postgresql.org/docs/18/auth-oauth.html) — PostgreSQL 18 OAuth authentication reference

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*