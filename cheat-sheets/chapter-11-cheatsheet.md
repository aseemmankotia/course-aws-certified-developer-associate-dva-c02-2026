## Chapter 11 Cheat Sheet — Diagnostics & Optimization

### Lambda Error Types by Invocation Model
| Invocation | Function Error Behavior | Throttling (429) Behavior |
|---|---|---|
| Synchronous (API GW, ALB) | Caller receives error immediately, NO auto-retry | Caller must retry |
| Asynchronous (S3, SNS, EventBridge) | Retries 2x (3 total attempts), then DLQ/Destination | Auto-retried with backoff |
| Event source mapping (Kinesis/DDB Streams) | Poller retries per shard until success or record expiry — blocks shard if unresolved | Poller backs off and retries |

### DLQ vs On-Failure Destinations
| Feature | DLQ | Destinations (newer) |
|---|---|---|
| Targets | SQS, SNS only | SQS, SNS, Lambda, EventBridge |
| Scope | Async invocation only | Async invocation (on-success + on-failure) |
| Context included | Payload only | Payload + invocation metadata (request ID, error) |
| Stream ESM support | ❌ | ✅ (SQS/SNS only for streams) |

### API Gateway Integration Errors
| Code | Meaning | Root Cause |
|---|---|---|
| 502 | Bad Gateway | Malformed Lambda proxy response (missing statusCode/body not string) |
| 504 | Integration Timeout | Lambda exceeded API GW integration timeout (max 29s, hard limit) |

### DynamoDB Throttling / Hot Partitions
- Symptom: `ProvisionedThroughputExceededException` despite low aggregate consumption.
- Diagnose: CloudWatch `ConsumedReadCapacityUnits`/`ConsumedWriteCapacityUnits` + Contributor Insights for DynamoDB.
- Fix: redesign partition key for higher cardinality; adaptive capacity helps but doesn't replace good key design; on-demand mode absorbs spikes but not structural hot spots.

### Caching: DAX vs ElastiCache
| | DAX | ElastiCache (Redis/Memcached) |
|---|---|---|
| Scope | DynamoDB only | General purpose |
| API | Drop-in DynamoDB SDK compatible | Separate client library |
| Latency | Microseconds | Sub-millisecond to low ms |
| Use case | Read-heavy DynamoDB workloads | Sessions, arbitrary app caching, pub/sub |

### Lambda Performance/Cost Tuning
- Memory range: 128 MB – 10,240 MB (1 MB increments).
- Timeout max: 900 seconds (15 min).
- CPU scales with memory — full vCPU at 1,769 MB.
- Billing = GB-seconds — more memory can lower cost if duration drops proportionally more.
- Reuse SDK clients/DB connections OUTSIDE handler for warm-start efficiency.

### Async Invocation Retry Config
- Default retries: 2 (3 total attempts).
- MaximumRetryAttempts: 0–2 (configurable).
- MaximumEventAgeInSeconds: 60s–6hrs (default 6hrs).

### Exam Traps
- DLQ ≠ works for stream-based event source mappings (use ESM's own on-failure destination, SQS/SNS only).
- 502 vs 504 confusion — malformed response vs timeout.
- Assuming more provisioned/on-demand DynamoDB capacity fixes hot partitions — it doesn't; fix key design.
- Assuming increasing Lambda memory always raises cost — it can lower cost via shorter duration.
