## Chapter 11: Root-Cause Diagnostics and Performance/Cost Optimization — Practice Questions

**Q1.** A Lambda function is invoked asynchronously by S3 event notifications. Failed events must be automatically retried twice, then routed to SQS for reprocessing AND trigger an alert via EventBridge. Which approach BEST meets these requirements?

A) Configure a Dead-Letter Queue (DLQ) pointing to SQS only
B) Configure two DLQs, one pointing to SQS and one pointing to EventBridge
C) Configure the on-failure destination as EventBridge, then create EventBridge rules to route the failure event to SQS and to an alerting target
D) Configure the on-failure destination as SQS, which automatically triggers EventBridge on message arrival

<details>
<summary>Answer</summary>

**Correct: C**

A function supports only one on-failure destination, but EventBridge can fan out to multiple downstream targets (SQS, alerting) via rules, and Destinations retain full invocation context unlike DLQ.

**Why the others are wrong:**
- DLQ only supports SQS/SNS directly, has no built-in fan-out or alerting mechanism, and includes less context than Destinations.
- Lambda only allows one DLQ configuration per function, not two.
- SQS does not automatically trigger EventBridge; no such native integration exists.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** API Gateway returns HTTP 504 for a Lambda proxy integration during peak traffic, though CloudWatch Logs show the Lambda function completing successfully after 12 seconds. What is the MOST likely root cause?

A) The Lambda function returned a malformed proxy response missing statusCode
B) The API Gateway integration timeout is configured lower than the Lambda execution time, so API Gateway times out before receiving the response
C) The Lambda function's reserved concurrency was set to 0
D) The Lambda function throttled due to exceeding account concurrency limits

<details>
<summary>Answer</summary>

**Correct: B**

A 504 indicates the integration timed out waiting for a response; the function did eventually complete, meaning API Gateway's timeout was shorter than the execution time.

**Why the others are wrong:**
- A malformed proxy response produces a 502, not a 504.
- Reserved concurrency of 0 would prevent invocation entirely — the function could not have completed successfully.
- Throttling would prevent the function from running at all, contradicting the successful completion shown in the logs.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A DynamoDB table uses a partition key of 'OrderDate' (e.g., '2026-01-15') to store millions of orders per day. The application experiences frequent ProvisionedThroughputExceededException errors even though aggregate consumed capacity is well below provisioned capacity. Which action BEST resolves this?

A) Enable DynamoDB auto scaling to increase provisioned throughput
B) Redesign the partition key to include a higher-cardinality attribute, such as OrderID appended to OrderDate, to distribute writes across more partitions
C) Switch the table to on-demand capacity mode only
D) Add a Global Secondary Index using the same partition key

<details>
<summary>Answer</summary>

**Correct: B**

The root cause is a hot partition from low-cardinality keys; redistributing writes via a higher-cardinality key fixes the underlying design flaw regardless of capacity mode.

**Why the others are wrong:**
- Auto scaling raises aggregate throughput but a single overloaded partition is still capped at its own per-partition limit.
- On-demand mode helps absorb spikes via adaptive capacity but does not fully resolve a structurally hot partition.
- A GSI with the same partition key inherits the identical hot partition problem.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A serverless order-processing app performs repeated GetItem calls against the same frequently accessed DynamoDB items and needs microsecond-level read latency without switching to a separate cache client library. Which caching solution should be used?

A) Amazon ElastiCache for Redis
B) Amazon DynamoDB Accelerator (DAX)
C) Amazon CloudFront
D) Lambda function-level in-memory caching using global variables

<details>
<summary>Answer</summary>

**Correct: B**

DAX is purpose-built for DynamoDB, API-compatible with existing DynamoDB SDK calls, and delivers microsecond read latency without code restructuring.

**Why the others are wrong:**
- ElastiCache is general-purpose and requires separate client code and cache-management logic.
- CloudFront caches HTTP content at the edge, not database query results.
- In-memory global variables only persist within a single warm container, are inconsistent across concurrent executions, and are unreliable at scale.

</details>

---

**Q5.** A team wants a single general-purpose in-memory caching layer to cache both repeated analytics query results and web session data, with support for multiple data structures and pub/sub messaging. Which service should they choose?

A) DAX with TTL configured
B) API Gateway caching
C) Amazon S3 with Transfer Acceleration
D) Amazon ElastiCache for Redis

<details>
<summary>Answer</summary>

**Correct: D**

Redis provides versatile data structures, pub/sub, and general-purpose caching applicable to both analytics results and session data.

**Why the others are wrong:**
- DAX only accelerates DynamoDB API calls, not arbitrary query result caching.
- API Gateway caching only caches HTTP method responses at the API layer, not custom session or query data.
- S3 is object storage and does not provide in-memory caching or pub/sub.

</details>

---

**Q6.** A developer increases a Lambda function's memory from 512 MB to 1,769 MB. What additional resource change occurs automatically?

A) The function's timeout automatically increases proportionally
B) The function's reserved concurrency automatically increases
C) The function automatically enables provisioned concurrency
D) The function is allocated the equivalent of one full vCPU, since Lambda allocates CPU proportionally to memory

<details>
<summary>Answer</summary>

**Correct: D**

Lambda scales CPU power linearly with memory allocation, and 1,769 MB is the threshold at which a function receives one full vCPU equivalent.

**Why the others are wrong:**
- Timeout is configured independently and is unaffected by memory changes (max 900 seconds).
- Concurrency limits are set separately and are not tied to memory configuration.
- Provisioned concurrency is a distinct opt-in feature that must be explicitly configured.

</details>

---

**Q7.** A Lambda function processing records from a DynamoDB Stream fails consistently on a single malformed record, causing the event source mapping to retry indefinitely and blocking newer records in that shard. Which configuration change resolves this without redeploying code?

A) Configure a Dead-Letter Queue (DLQ) directly on the Lambda function
B) Increase the Lambda function's timeout to allow more processing time per invocation
C) Decrease the batch size to 1 record per invocation
D) Configure MaximumRetryAttempts and an on-failure destination (SQS or SNS) on the event source mapping so failed batches are set aside and processing continues

<details>
<summary>Answer</summary>

**Correct: D**

Stream-based event source mappings have their own retry limit and failure destination settings (SQS/SNS only), which bypass the poison record and unblock the shard.

**Why the others are wrong:**
- A DLQ configured on the function applies only to asynchronous invocations, not stream-based event source mappings.
- Increasing timeout does not fix a function error caused by malformed data.
- Reducing batch size may isolate the record but does not stop indefinite retries; a failure destination configuration is required.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** Which statement correctly differentiates Lambda function errors from Lambda throttling errors?

A) Function errors always trigger automatic retries; throttling errors are never retried
B) Function errors occur when code throws an exception or times out; throttling errors (429) occur when concurrent execution limits are exceeded, and are automatically retried by Lambda only for asynchronous invocations
C) Function errors and throttling errors are retried identically regardless of invocation type
D) Throttling errors only occur with synchronous invocations, and function errors only occur with asynchronous invocations

<details>
<summary>Answer</summary>

**Correct: B**

Function errors stem from code exceptions/timeouts; throttling errors (429) occur from exceeding concurrency limits. For async invocations, Lambda automatically retries both error types; for sync invocations, the caller must handle retries.

**Why the others are wrong:**
- Synchronous invocation function errors are NOT automatically retried by Lambda.
- Retry behavior differs significantly by invocation type (sync, async, event source mapping).
- Both error types can occur under either invocation type.

</details>

---

