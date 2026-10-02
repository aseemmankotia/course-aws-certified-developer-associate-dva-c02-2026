## Chapter 2: Amazon API Gateway: Integrations, Authorizers, and Throttling — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Default API Gateway account-level throttle limits per Region? | 10,000 requests/sec steady-state, 5,000 requests burst (token bucket). Can be raised via a service quota increase. Exam tests this exact pair of numbers. |
| 2 | Max integration timeout for both REST API and HTTP API backend integrations? | 29 seconds. If your Lambda or backend takes longer, the client gets a 504 regardless of the Lambda's own timeout setting. |
| 3 | Max payload size for API Gateway request/response (REST and HTTP API)? | 10 MB. Exceeding this returns a 413 Payload Too Large error — exam scenario often involves large file uploads needing S3 pre-signed URLs instead. |
| 4 | REST API vs HTTP API: which supports usage plans and API keys? | REST API only. HTTP APIs have no usage plans, no API keys — for third-party rate limiting/monetization you must use REST API. |
| 5 | REST API vs HTTP API: which supports response caching and AWS WAF? | REST API only. HTTP APIs do not support caching or WAF integration. Choose REST if these are requirements. |
| 6 | Lambda proxy integration vs Lambda custom (non-proxy) integration — key difference? | Proxy: entire raw request passed as event; Lambda must return {statusCode, headers, body}. Custom: uses VTL mapping templates to transform request/response, no fixed response shape required. |
| 7 | Lambda authorizer TOKEN type vs REQUEST type — what's the difference? | TOKEN: receives only a single bearer token (e.g., Authorization header) for validation. REQUEST: receives headers, query string params, stage variables, and context — used for multi-parameter auth logic. |
| 8 | Cognito user pool authorizer vs IAM authorization (SigV4) — when to use each? | Cognito: end-user/human authentication (mobile/web app sign-in). IAM SigV4: service-to-service or AWS SDK calls where the caller has an IAM identity/role — no custom auth code needed. |
| 9 | How do you route one API stage to different Lambda aliases per environment (dev/prod)? | Use stage variables referenced inside the integration URI, e.g., arn:...function:myFunction:${stageVariables.lambdaAlias}. One API definition, environment-specific routing. |
| 10 | What VTL command extracts a value from the incoming JSON request body in a mapping template? | $input.path('$.fieldName') for JSON, or $input.json('$.fieldName') to get raw JSON string. Used only in REST API custom (non-proxy) integrations — HTTP APIs don't support VTL mapping templates. |
| 11 | Usage plan components — what two settings does a usage plan combine? | Throttle limits (rate + burst per API key) and Quota (requests per day/week/month). Attached to one or more API keys to enforce per-customer rate limiting on REST APIs. |
| 12 | REST API vs HTTP API — relative request pricing? | REST API: $3.50 per million requests (first 333M/month). HTTP API: $1.00 per million requests (first 300M/month) — up to ~70% cheaper, plus lower latency. Exam tests choosing HTTP API when cost/latency matter and full feature set isn't needed. |
