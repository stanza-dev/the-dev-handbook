---
source_course: "nestjs-security"
source_lesson: "nestjs-security-audit-logging"
---

# Security Audit Logging

## Introduction

Audit logs record security-relevant events for compliance, forensics, and monitoring. They answer "who did what, when, and from where?" Essential for detecting breaches, investigating incidents, and meeting regulatory requirements.

## Key Concepts

- **Audit Trail**: Chronological record of system activities
- **Non-repudiation**: Users can't deny their actions
- **Immutability**: Logs can't be modified after creation
- **Correlation ID**: Links related events across services

## Real World Context

Audit logging required for:
- GDPR, HIPAA, SOC2 compliance
- Breach investigation and forensics
- User activity monitoring
- Security incident detection

## Deep Dive

### Audit Log Entity

```typescript
@Entity()
export class AuditLog {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  action: string; // 'USER_LOGIN', 'PASSWORD_CHANGE', etc.

  @Column({ nullable: true })
  userId: string;

  @Column({ nullable: true })
  targetId: string; // ID of affected resource

  @Column()
  targetType: string; // 'User', 'Order', etc.

  @Column('jsonb', { nullable: true })
  metadata: Record<string, any>;

  @Column()
  ipAddress: string;

  @Column({ nullable: true })
  userAgent: string;

  @Column()
  correlationId: string;

  @Column({ type: 'enum', enum: ['SUCCESS', 'FAILURE'] })
  outcome: 'SUCCESS' | 'FAILURE';

  @CreateDateColumn()
  createdAt: Date;
}
```

### Audit Service

```typescript
@Injectable()
export class AuditService {
  constructor(
    @InjectRepository(AuditLog)
    private auditRepo: Repository<AuditLog>,
  ) {}

  async log(entry: CreateAuditLogDto) {
    await this.auditRepo.save({
      ...entry,
      createdAt: new Date(),
    });
  }

  async logSecurityEvent(
    action: string,
    request: Request,
    outcome: 'SUCCESS' | 'FAILURE',
    metadata?: Record<string, any>,
  ) {
    await this.log({
      action,
      userId: request.user?.id,
      ipAddress: request.ip,
      userAgent: request.headers['user-agent'],
      correlationId: request.headers['x-correlation-id'] as string,
      outcome,
      metadata,
    });
  }
}
```

### Audit Decorator and Interceptor

```typescript
export const Audit = (action: string) =>
  SetMetadata('audit:action', action);

@Injectable()
export class AuditInterceptor implements NestInterceptor {
  constructor(
    private reflector: Reflector,
    private auditService: AuditService,
  ) {}

  intercept(context: ExecutionContext, next: CallHandler) {
    const action = this.reflector.get<string>('audit:action', context.getHandler());
    if (!action) return next.handle();

    const request = context.switchToHttp().getRequest();
    const startTime = Date.now();

    return next.handle().pipe(
      tap(async () => {
        await this.auditService.logSecurityEvent(action, request, 'SUCCESS', {
          duration: Date.now() - startTime,
        });
      }),
      catchError(async (error) => {
        await this.auditService.logSecurityEvent(action, request, 'FAILURE', {
          error: error.message,
          duration: Date.now() - startTime,
        });
        throw error;
      }),
    );
  }
}

// Usage
@Audit('USER_DELETE')
@Delete(':id')
deleteUser(@Param('id') id: string) { ... }
```

### Critical Events to Log

```typescript
// Authentication events
@Audit('USER_LOGIN')
@Audit('USER_LOGOUT')
@Audit('LOGIN_FAILED')
@Audit('PASSWORD_CHANGE')
@Audit('PASSWORD_RESET_REQUEST')
@Audit('MFA_ENABLED')
@Audit('MFA_DISABLED')

// Authorization events
@Audit('PERMISSION_DENIED')
@Audit('ROLE_ASSIGNED')
@Audit('ROLE_REMOVED')

// Data events
@Audit('USER_CREATED')
@Audit('USER_DELETED')
@Audit('SENSITIVE_DATA_ACCESSED')
@Audit('DATA_EXPORTED')

// Admin events
@Audit('SETTINGS_CHANGED')
@Audit('API_KEY_CREATED')
@Audit('API_KEY_REVOKED')
```

### Log Retention and Protection

```typescript
@Injectable()
export class AuditRetentionService {
  @Cron('0 0 * * *') // Daily
  async enforceRetention() {
    const retentionDays = this.configService.get('AUDIT_RETENTION_DAYS', 365);
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - retentionDays);

    // Archive old logs before deletion
    await this.archiveOldLogs(cutoffDate);
    
    // Delete archived logs
    await this.auditRepo.delete({
      createdAt: LessThan(cutoffDate),
      archived: true,
    });
  }
}
```

## Common Pitfalls

1. **Logging sensitive data**: Don't log passwords, tokens, or PII.
2. **Mutable logs**: Logs should be append-only.
3. **No correlation**: Without correlation IDs, related events can't be linked.

## Best Practices

- Log all authentication and authorization events
- Include correlation IDs for request tracing
- Never log sensitive data (passwords, tokens, PII)
- Store logs separately from application data
- Implement retention policies for compliance
- Monitor logs for suspicious patterns

## Summary

Audit logging records security-relevant events. Log authentication, authorization, and sensitive data access. Include correlation IDs, user context, and IP addresses. Store logs securely with appropriate retention policies.

## Resources

- [Logger](https://docs.nestjs.com/techniques/logger) — Logging best practices

---

> 📘 *This lesson is part of the [NestJS Security](https://stanza.dev/courses/nestjs-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*