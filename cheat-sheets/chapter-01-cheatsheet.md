## Lambda Cheat Sheet — DVA-C02

### Invocation Models
| Model | Trigger Examples | Behavior |
|---|---|---|
| Synchronous (RequestResponse) | API Gateway, ALB, direct SDK Invoke | Caller waits for response; errors returned directly to caller |
| Asynchronous (Event) | S3, SNS, EventBridge | Caller gets 202 immediately; Lambda retries **2x** on failure; use DLQ or on-failure destination to capture unresolved failures; events queued up to 6 hrs (configurable MaximumEventAgeInSeconds) |
| Poll-based (Event Source Mapping) | SQS, Kinesis, DynamoDB Streams, MSK/Kafka | Lambda service polls source, batches records, invokes synchronously on your behalf; scaling tied to shards/queue depth |

### Concurrency
| Type | Purpose | Key Behavior |
|---|---|---|
| Reserved Concurrency | Sets a hard max AND guarantees capacity | Reduces account's shared unreserved pool (default 1,000 minus reservations); setting to 0 disables function |
| Provisioned Concurrency | Pre-warms environments | Eliminates cold starts for pre-initialized instances; works with Application Auto Scaling; billed separately even when idle |

### Limits (memorize exact numbers)
| Setting | Value |
|---|---|
| Timeout: default / max | 3 sec / **900 sec (15 min)** |
| Memory range | **128 MB – 10,240 MB** (1 MB increments) |
| Ephemeral storage (/tmp) | 512 MB default, up to 10,240 MB configurable |
| .zip package: zipped / unzipped (incl. layers) | 50 MB / **250 MB** |
| Container image max size | **10 GB** |
| Max layers per function | **5** |
| Env variables total size | **4 KB** |
| Async retry attempts | 2 retries (3 total attempts) |
| Async event max retention | up to 6 hours |

### Packaging Decision Rule
- Small function, few deps → **.zip**
- Shared libs across many functions → **Layers**
- Deps/custom runtime > 250 MB unzipped → **Container image** (up to 10 GB)

### Environment Variables & Encryption
- Encrypted at rest by default with AWS managed key.
- For sensitive values (secrets, credentials): specify a **customer managed CMK**, optionally use console encryption helpers to encrypt client-side before storage.
- Prefer Secrets Manager/Parameter Store for rotation-sensitive secrets over plain env vars.

### Cold Starts
- Occur on first invocation of a new execution environment (init phase: runtime bootstrap + handler init code).
- Mitigated by **Provisioned Concurrency**, not by Reserved Concurrency or bigger timeout.
- Container images and large packages can increase cold start time.

### CLI Quick Reference
```bash
# Reserved concurrency
aws lambda put-function-concurrency --function-name myFunc --reserved-concurrent-executions 10

# Provisioned concurrency
aws lambda put-provisioned-concurrency-config --function-name myFunc --qualifier 1 --provisioned-concurrent-executions 5

# Publish a layer
aws lambda publish-layer-version --layer-name shared-libs --zip-file fileb://layer.zip
```

### Common Exam Traps
- Confusing reserved (capacity ceiling) with provisioned (pre-warmed) concurrency.
- Assuming S3/SNS triggers are synchronous — they are **asynchronous**.
- Assuming SQS/Kinesis triggers are asynchronous — they are **poll-based event source mappings**.
- Forgetting the 250 MB unzipped limit includes layers, not just function code.
