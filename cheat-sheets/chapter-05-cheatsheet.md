# Chapter 5 Cheat Sheet: IAM, STS, Cognito

## Identity-Based vs Resource-Based Policies
| Feature | Identity-Based | Resource-Based |
|---|---|---|
| Attached to | User / Group / Role | Resource (S3, KMS, SQS, Lambda) |
| Principal element | Never present | Required |
| Cross-account access | Needs role assumption | Grants directly, no assumption needed |
| Combined evaluation | Union of Allow, unless explicit Deny anywhere | Same |

## IAM Policy Evaluation Order
1. **Explicit Deny** (any policy type) → wins, always
2. **Explicit Allow** → grants access if no deny found
3. **Implicit Deny** → default if nothing matches

**Exam trap:** Students assume identity-based Allow overrides resource-based Deny. It never does — Deny always wins.

## IAM Users vs Roles
| | IAM User | IAM Role |
|---|---|---|
| Credentials | Long-term (access key/secret) | Temporary, issued on assumption |
| Use case | Rare human access | Workloads, cross-account, federation |
| Best practice | Avoid for apps | Preferred for EC2/Lambda/ECS |

## STS AssumeRole Family
| API | Use Case | Max Duration |
|---|---|---|
| AssumeRole | Cross-account / same-account role switch | Up to role's MaxSessionDuration (max 12h/43200s) |
| AssumeRoleWithWebIdentity | OIDC providers (Cognito, Google, Facebook) | Up to 12h |
| AssumeRoleWithSAML | Corporate SAML 2.0 IdP (AD FS) | Up to 12h |
| GetSessionToken | MFA-protected IAM user session | Default 12h, max 36h (129600s) |

- Default AssumeRole session (if DurationSeconds omitted): **3600s (1h)**
- Minimum: **900s (15 min)**

## Cognito User Pools vs Identity Pools
| | User Pool | Identity Pool |
|---|---|---|
| Purpose | Authentication (directory) | Authorization (AWS access) |
| Output | JWT (ID, Access, Refresh tokens) | Temporary AWS credentials via STS |
| Supports | Sign-up/in, MFA, password policy | Federates User Pool, social IdPs, SAML, unauthenticated (guest) |
| Token validity (ID/Access) | 5 min – 24h (default 1h) | N/A |
| Refresh token validity | 60 min – 10 years (default 30 days) | N/A |

## API Gateway + Cognito
- **Cognito User Pool Authorizer**: native, validates JWT signature/expiry automatically — no custom code.
- **Lambda Authorizer**: custom logic, use when validation rules are non-standard.
- **IAM Authorizer**: validates SigV4-signed requests, not JWTs.
- Client sends: `Authorization: Bearer <id_token>`

## Common Gotchas
- Cross-account S3 access needs BOTH an identity policy Allow AND a resource (bucket) policy Allow.
- Requesting `DurationSeconds` above the role's `MaxSessionDuration` → AssumeRole call fails.
- Identity Pool ≠ authentication; it converts an already-authenticated identity into AWS credentials.
- Explicit Deny in ANY policy (identity, resource, SCP, permissions boundary) always overrides Allow.
