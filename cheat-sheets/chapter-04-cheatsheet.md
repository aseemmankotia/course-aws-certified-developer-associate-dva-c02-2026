## Chapter 4 Cheat Sheet — S3 & Application Integration

### S3 Presigned URLs & Multipart Upload
| Feature | Value |
|---|---|
| Max presigned URL expiration | 7 days (SigV4, all credential types) |
| Multipart part size | 5MB–5GB per part (last part can be <5MB) |
| Max parts per upload | 10,000 |
| Max object size (multipart) | 5TB |
| When to use multipart | Recommended >100MB, required >5GB |
| S3 Event Notifications | Trigger Lambda/SQS/SNS on PUT, POST, COPY, DELETE, multipart complete |

### SQS Standard vs FIFO
| | Standard | FIFO |
|---|---|---|
| Delivery | At-least-once | Exactly-once (dedup) |
| Order | Best-effort | Strict, per MessageGroupId |
| Throughput | Nearly unlimited | 300/sec (no batch), 3,000/sec (batch), higher w/ High Throughput mode |
| Queue name suffix | none | must end in `.fifo` |
| Max message size | 256 KB (both) | 256 KB |
| Visibility timeout | Default 30s, max 12h | same |
| Retention | Default 4d, max 14d | same |
| Long polling max wait | 20s | 20s |

### SNS
- Pub/sub fan-out: one message → many independent subscribers (SQS, Lambda, HTTP/S, email, mobile push).
- Max message size: 256 KB.
- Supports FIFO topics (paired with SQS FIFO) and message filtering policies per subscription.

### EventBridge
- Default bus (AWS service events), custom buses (your app events), partner buses (SaaS integrations).
- Rules match event patterns (content-based) and route to targets (Lambda, Step Functions, SQS, SNS, Kinesis, etc.).
- Schema registry: infer/validate event structure; generate code bindings.
- PutEvents entry max size: 256 KB.
- Failed target invocations retry per rule's retry policy, with optional DLQ.

### Step Functions
| | Standard | Express |
|---|---|---|
| Max duration | 1 year | 5 minutes |
| Semantics | Exactly-once | At-least-once (async) / At-most-once (sync) |
| Pricing | Per state transition | Per execution + duration + memory |
| Use case | Long-running, auditable, human approvals | High-volume, short, event processing |
| History in console | Full execution history | Limited (CloudWatch Logs) |

### Decision Matrix (heavily tested)
- **One consumer, decouple producer/consumer** → SQS.
- **Many independent consumers, same message** → SNS fan-out.
- **Content-based routing, multiple sources/accounts, schema needed** → EventBridge.
- **Multi-step orchestration with state, branching, retries, human wait** → Step Functions.
- **Strict order + no duplicates needed** → SQS FIFO (group by logical entity, e.g., customer/order ID).

### Idempotency & Retries
- Use exponential backoff + jitter for retries (AWS SDK default behavior).
- Design consumers to be idempotent (dedupe by message ID / idempotency key) since SQS Standard and most async integrations are at-least-once.
- SQS FIFO dedup: content-based deduplication or explicit `MessageDeduplicationId`, 5-minute dedup interval.

### Common Exam Traps
- Assuming SQS Standard preserves order — it does not.
- Confusing SNS (push fan-out) with SQS (pull, one consumer per message).
- Thinking presigned URLs can exceed 7 days.
- Forgetting Step Functions Express 5-minute cap when a scenario mentions long waits.
