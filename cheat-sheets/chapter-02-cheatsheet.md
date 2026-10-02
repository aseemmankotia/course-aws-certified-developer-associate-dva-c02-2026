## API Gateway Cheat Sheet — DVA-C02

### REST API vs HTTP API
| Feature | REST API | HTTP API |
|---|---|---|
| Usage plans / API keys | Yes | No |
| Request validation (schema) | Yes | No |
| Response caching | Yes | No |
| AWS WAF integration | Yes | No |
| Edge-optimized endpoint | Yes | No (regional only) |
| Private integrations (VPC Link) | Yes | Yes |
| JWT authorizer (native) | No | Yes |
| Lambda authorizer | Yes (TOKEN/REQUEST) | Yes (simple format only) |
| IAM (SigV4) authorization | Yes | Yes |
| Pricing (per million req) | $3.50 (first 333M) | $1.00 (first 300M) |
| Latency | Higher | Lower (~60% less overhead) |

### Lambda Integration Types
| Type | Request Handling | Response Handling |
|---|---|---|
| Lambda Proxy | Raw event passed as-is | Lambda must return {statusCode, headers, body} |
| Lambda Custom (non-proxy) | VTL mapping template transforms request | VTL mapping template transforms response |

**Exam trap:** In proxy integration, any mapping template configured is ignored — full raw request/response only.

### Authorizer Types
| Type | Use Case | Notes |
|---|---|---|
| Lambda Authorizer (TOKEN) | Single bearer token validation | Custom logic, returns IAM policy |
| Lambda Authorizer (REQUEST) | Multi-param auth (headers, query, context) | Custom logic, returns IAM policy |
| Cognito User Pool Authorizer | End-user/human auth | REST API: Cognito authorizer type. HTTP API: JWT authorizer w/ Cognito as issuer |
| IAM (SigV4) | AWS SDK / service-to-service calls | No custom code; IAM policy on caller + resource policy |

### Throttling
- **Account-level default:** 10,000 RPS steady-state / 5,000 RPS burst (per Region, token bucket) — can request increase via Service Quotas.
- **Stage/method-level:** Set lower than account default via stage settings or usage plans; cannot exceed account ceiling without increase.
- **Usage plans:** Combine throttle (rate + burst per key) + quota (requests/day, week, or month) — REST API only.

### Stages & Stage Variables
- Stages = named deployments (dev, test, prod) with independent settings (throttling, caching, logging).
- Stage variables = key/value pairs referenced as `${stageVariables.varName}` inside integration URIs — used to route to different Lambda aliases/backends per stage without duplicating the API.
- Deployment history is tracked per API; a stage always points to exactly one deployment at a time.

### Mapping Templates (VTL) — REST API custom integration only
- `$input.path('$.field')` — extract JSON value
- `$input.json('$.field')` — extract raw JSON string
- `$context.*` — request context (requestId, identity, stage)
- `$stageVariables.*` — access stage variables in template
- **Not supported** in HTTP APIs (only basic parameter mapping) or Lambda proxy integration.

### Key Numbers
- Max payload: 10 MB (REST & HTTP API)
- Max integration timeout: 29 seconds (both types)
- Cache capacity (REST only): 0.5 GB–237 GB, default TTL 300s, max 3600s
