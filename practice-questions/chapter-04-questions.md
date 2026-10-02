## Chapter 4: Amazon S3 and Application Integration: SQS, SNS, EventBridge, Step Functions — Practice Questions

**Q1.** A ride-sharing app must process ride-request events per driver in the exact order they occur, and must guarantee that a duplicate network retry does not create two ride records. Which SQS configuration BEST meets these requirements?

A) SQS Standard queue with a Lambda trigger and a DynamoDB conditional write
B) SQS Standard queue with visibility timeout set to 12 hours
C) SQS FIFO queue using MessageGroupId set to the driver ID, with content-based deduplication enabled
D) SNS FIFO topic fanning out to multiple SQS Standard queues

<details>
<summary>Answer</summary>

**Correct: C**

FIFO queues guarantee exactly-once processing and strict ordering within a MessageGroupId. Using the driver ID as the group ID ensures per-driver ordering while allowing parallelism across drivers, and content-based deduplication prevents duplicate ride records from retries.

**Why the others are wrong:**
- Standard queues offer best-effort ordering only — a conditional write reduces duplicate writes but doesn't guarantee delivery order.
- Increasing visibility timeout affects redelivery timing, not ordering or exactly-once semantics.
- SNS FIFO can preserve order to a subscriber, but fanning out to Standard SQS queues breaks the ordering guarantee at the consumer end.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An order-processing event must simultaneously trigger a Lambda function, land in an SQS queue for a legacy inventory system, and be archived to S3 — with the ability to add more subscribers later without changing the publisher. Which architecture is BEST?

A) Have the order service call each downstream system directly via its SDK
B) Write the event to a single SQS queue and have each consumer poll it independently
C) Publish the event to an SNS topic and subscribe Lambda, SQS, and a Firehose delivery stream to it
D) Use Step Functions to invoke each service sequentially in a state machine

<details>
<summary>Answer</summary>

**Correct: C**

SNS pub/sub fan-out lets one publish reach many independent subscribers (Lambda, SQS, Firehose→S3) without coupling the publisher to the number or type of consumers.

**Why the others are wrong:**
- Direct SDK calls tightly couple the publisher to every consumer, making it hard to add subscribers.
- A single SQS queue delivers each message to only ONE consumer (competing consumers), not all three simultaneously.
- Step Functions is for orchestrating a defined multi-step workflow, not for one-to-many fan-out to independent, decoupled subscribers.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A loan-approval process involves multiple sequential and parallel steps, human approval waits that can last days, automatic retries with backoff on failures, and a visual audit trail of every execution for compliance. Which service BEST fits this requirement?

A) Amazon EventBridge rule with multiple targets
B) AWS Step Functions Standard Workflow
C) AWS Step Functions Express Workflow
D) Amazon SQS with a Lambda consumer and CloudWatch Logs

<details>
<summary>Answer</summary>

**Correct: B**

Standard Workflows support executions up to one year, exactly-once execution semantics, built-in retry/catch with backoff, and a visual console with full execution history — ideal for long-running, auditable, human-in-the-loop processes.

**Why the others are wrong:**
- EventBridge routes events to targets but has no native state, retries-with-backoff configuration per step, or wait states — it isn't a workflow orchestrator.
- Express Workflows are capped at 5 minutes execution duration and are at-least-once/at-most-once — unsuitable for multi-day human approval waits.
- SQS+Lambda requires you to build state tracking, retry logic, and audit trails yourself; it lacks a visual workflow and long-wait state support.

</details>

---

**Q4.** A company runs 20 AWS accounts and wants events from a partner SaaS billing system, plus custom application events, to be filtered by content and routed to different targets (Lambda, Step Functions, SQS) across accounts, validated against a defined schema. Which service should they use?

A) Amazon SNS with message filtering policies
B) Amazon EventBridge with custom event buses, rules, and schema registry
C) Amazon SQS FIFO with message attributes
D) AWS Step Functions with a Choice state

<details>
<summary>Answer</summary>

**Correct: B**

EventBridge supports partner/custom event buses, cross-account routing, content-based rule filtering, and a schema registry to validate and discover event structures — purpose-built for this scenario.

**Why the others are wrong:**
- SNS filtering works on subscription attributes but has no schema registry and weaker cross-account, multi-bus routing model than EventBridge.
- SQS is a point-to-point queue; it doesn't support content-based routing to multiple different target types across accounts.
- Step Functions orchestrates steps within a single execution; it isn't designed to ingest and route external partner events across accounts.

</details>

---

**Q5.** A mobile app needs users to upload 2GB video files directly to a private S3 bucket without the app backend proxying the bytes and without embedding AWS credentials in the app. Which combination of features should be used?

A) Make the bucket public and use standard PUT requests from the client
B) Generate a presigned URL for a multipart upload and have the client upload parts directly to S3
C) Store IAM access keys in the mobile app and call PutObject directly
D) Use an S3 Batch Operations job triggered from the client

<details>
<summary>Answer</summary>

**Correct: B**

Presigned URLs grant temporary, credential-free access scoped to specific S3 actions; combined with multipart upload (required/recommended for large objects), the client can upload directly to S3 in parallel parts without exposing credentials or routing bytes through the backend.

**Why the others are wrong:**
- Making the bucket public violates least privilege and exposes all objects, not just the intended upload.
- Embedding long-lived IAM credentials in a mobile app is a major security risk and against AWS best practices.
- S3 Batch Operations processes large numbers of objects already in S3 (e.g., bulk copy/tag) — it's not a client upload mechanism.

</details>

---

**Q6.** A team configures a standard SQS FIFO queue (no batching) and needs to plan capacity. What is the maximum sustained throughput they should design for?

A) Unlimited, same as Standard queues
B) 3,000 messages per second
C) 30 messages per second
D) 300 messages per second (or 300 API calls per second)

<details>
<summary>Answer</summary>

**Correct: D**

A standard FIFO queue without batching supports up to 300 API calls (SendMessage/ReceiveMessage/DeleteMessage) per second. Using batch APIs raises this to 3,000 messages/second, and High Throughput FIFO mode can go higher.

**Why the others are wrong:**
- Standard (non-FIFO) queues have nearly unlimited throughput — FIFO does not share this characteristic.
- 3,000 messages/second is only achievable when using batching (10 messages per batch API call), not with single-message calls.
- 30/sec understates the actual documented limit of 300/sec without batching.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A developer generates an S3 presigned URL using AWS Signature Version 4 signing to allow a partner to download a report. What is the MAXIMUM expiration time allowed for this presigned URL?

A) 12 hours
B) 24 hours
C) 7 days
D) 30 days

<details>
<summary>Answer</summary>

**Correct: C**

Presigned URLs signed with SigV4 have a maximum expiration of 7 days (604,800 seconds), regardless of whether IAM user or temporary (STS) credentials are used — and if STS credentials are used, the URL also can't outlive those credentials.

**Why the others are wrong:**
- 12 hours is well under the maximum and is a distractor value, not a real limit.
- 24 hours is a common but incorrect default assumption; it is not the maximum.
- 30 days exceeds the actual SigV4 hard limit of 7 days.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** Which statement correctly differentiates Amazon SQS Standard queues from FIFO queues?

A) Standard queues guarantee exactly-once processing; FIFO queues guarantee at-least-once processing
B) Both queue types guarantee strict message order but differ only in throughput limits
C) FIFO queues can only be used with Amazon SNS as the publisher, while Standard queues cannot
D) Standard queues provide at-least-once delivery with best-effort ordering; FIFO queues provide exactly-once processing with strict ordering

<details>
<summary>Answer</summary>

**Correct: D**

Standard queues deliver messages at least once and may occasionally deliver out of order or duplicated; FIFO queues preserve the exact order within a message group and enforce exactly-once processing via deduplication.

**Why the others are wrong:**
- This reverses the actual guarantees — Standard is at-least-once/unordered, not exactly-once.
- Standard queues explicitly do NOT guarantee order; only FIFO does, so this statement is false.
- Either queue type can be subscribed to an SNS topic (SNS also offers a FIFO topic type to pair with SQS FIFO), so this restriction doesn't exist.

</details>

---

