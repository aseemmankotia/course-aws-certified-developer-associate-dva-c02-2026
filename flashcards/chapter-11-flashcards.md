## Chapter 11: Root-Cause Diagnostics and Performance/Cost Optimization — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Lambda async invocation default retry count on function error? | 2 retries (3 total attempts) with delays between them. Exam tip: after retries exhaust, the event goes to DLQ/on-failure destination if configured, otherwise it's discarded. |
| 2 | DLQ supported targets for Lambda async invocations? | SQS or SNS only. Exam tip: On-failure Destinations support SQS, SNS, Lambda, and EventBridge — the newer, more flexible mechanism. |
| 3 | Event source mapping (Kinesis/DynamoDB Streams) on-failure destination supported targets? | SQS and SNS only (no Lambda or EventBridge). Exam tip: a function-level DLQ does NOT apply to stream-based event source mappings — configure the ESM's own failure destination instead. |
| 4 | API Gateway 502 vs 504 for Lambda proxy integration? | 502 = malformed proxy response (e.g., missing statusCode or non-string body). 504 = integration timeout — Lambda took longer than the configured integration timeout. |
| 5 | API Gateway maximum integration timeout? | 29 seconds — hard limit, not configurable higher. Exam tip: for longer processing, use async patterns (SQS, Step Functions) instead. |
| 6 | Lambda max memory and max timeout? | Memory: 128 MB–10,240 MB (1 MB increments). Timeout: max 900 seconds (15 minutes). |
| 7 | At what memory setting does Lambda allocate a full vCPU equivalent? | 1,769 MB. CPU scales proportionally with memory. Exam tip: raising memory can sometimes LOWER total cost if execution time drops enough (billed in GB-seconds). |
| 8 | How to diagnose a DynamoDB hot partition? | Use CloudWatch ConsumedReadCapacityUnits/ConsumedWriteCapacityUnits trends plus DynamoDB Contributor Insights to find frequently accessed keys. Exam tip: fix via higher-cardinality partition key design, not just more capacity. |
| 9 | DAX vs ElastiCache — when to use which? | DAX: DynamoDB-only, drop-in API-compatible, microsecond latency for DynamoDB reads. ElastiCache (Redis/Memcached): general-purpose caching for any app data, sessions, or non-DynamoDB workloads. |
| 10 | Best practice for SDK client reuse in Lambda? | Initialize SDK clients/DB connections OUTSIDE the handler (global/module scope) so they persist across warm invocations in the same execution context, avoiding reconnection overhead. |
| 11 | Lambda throttling error code and cause? | HTTP 429 TooManyRequestsException — triggered when concurrent execution limits are exceeded. Async invocations auto-retry; sync invocations require caller-side retry logic. |
| 12 | Maximum event age for Lambda async retries? | Configurable 60 seconds to 6 hours (default 6 hours). Exam tip: once this window or MaximumRetryAttempts (0-2) is exceeded, the event is discarded or sent to DLQ/on-failure destination. |
