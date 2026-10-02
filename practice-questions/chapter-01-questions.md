## Chapter 1: AWS Lambda: Concurrency, Packaging, and Event Sources — Practice Questions

**Q1.** A team sets reserved concurrency of 50 on a critical payment-processing Lambda function. Shortly after, other unrelated Lambda functions in the same account start throwing Throttled errors during peak traffic. What is the MOST likely cause?

A) The payment function's reserved concurrency reduced the pool of unreserved concurrency available to all other functions in the account
B) Reserved concurrency automatically applies to every function in the account
C) Lambda functions share a global memory limit that was exceeded
D) The payment function's provisioned concurrency consumed all available warm environments

<details>
<summary>Answer</summary>

**Correct: A**

Reserved concurrency carves out and guarantees capacity for one function, subtracting it from the account's unreserved concurrency pool (default 1,000 minus reserved amounts). If enough is reserved, other functions can be throttled when the shared pool is exhausted.

**Why the others are wrong:**
- Reserved concurrency is set per function and does not propagate to other functions
- Lambda memory limits are per-invocation, not a shared account-wide ceiling
- The scenario describes reserved concurrency, not provisioned concurrency, and provisioned concurrency does not remove capacity from the shared pool the same way

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An S3 bucket triggers a Lambda function to process uploaded images. The function occasionally fails due to a downstream API being unavailable. The team needs failed events captured for later inspection without writing custom retry logic. What should they configure?

A) Increase the function's synchronous timeout to 15 minutes
B) Set reserved concurrency to 0 to force retries
C) Configure an on-failure destination (or DLQ) for the asynchronous invocation
D) Switch the trigger to an event source mapping with a batch window

<details>
<summary>Answer</summary>

**Correct: C**

S3-to-Lambda is an asynchronous invocation. Lambda automatically retries twice, then discards the event unless a dead-letter queue or on-failure (Destinations) target is configured to capture it for later inspection.

**Why the others are wrong:**
- Timeout controls per-invocation duration, not failure capture
- Setting reserved concurrency to 0 disables the function entirely, blocking all invocations
- S3 does not support event source mappings; that mechanism is used for poll-based sources like SQS, Kinesis, and DynamoDB Streams

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A latency-sensitive API Gateway-backed Lambda function experiences unacceptable p99 latency spikes only during sudden traffic bursts, caused by cold starts. Which configuration BEST resolves this?

A) Increase reserved concurrency to match peak traffic
B) Increase the function timeout to 15 minutes
C) Convert the deployment package from .zip to a container image
D) Enable provisioned concurrency with Application Auto Scaling

<details>
<summary>Answer</summary>

**Correct: D**

Provisioned concurrency keeps a specified number of execution environments pre-initialized, eliminating the cold-start init phase. Combined with Application Auto Scaling, it can scale up ahead of predictable traffic bursts.

**Why the others are wrong:**
- Reserved concurrency only sets a concurrency ceiling/guarantee; it does not pre-warm environments and does not eliminate cold starts
- Timeout controls how long an invocation may run, unrelated to cold start latency
- Container images do not inherently reduce cold start latency and can sometimes increase it depending on image size

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A developer needs to process messages from an SQS standard queue using Lambda, with Lambda controlling batching and polling automatically. Which invocation model applies?

A) Poll-based invocation via event source mapping
B) Synchronous invocation (RequestResponse)
C) Asynchronous invocation (Event)
D) Streaming invocation via Kinesis Data Streams

<details>
<summary>Answer</summary>

**Correct: A**

SQS is not capable of invoking Lambda directly; instead, Lambda's service uses an event source mapping to poll the queue, batch messages, and invoke the function synchronously on your behalf.

**Why the others are wrong:**
- Synchronous invocation is caller-initiated (e.g., API Gateway), not a polling mechanism
- Asynchronous invocation applies to services like S3 and SNS that push events directly to Lambda
- Kinesis is a separate poll-based source; SQS is not part of Kinesis

</details>

---

**Q5.** A function's deployment package, including all dependencies and layers, has grown to 400 MB unzipped, exceeding the .zip deployment limit. Which packaging option should be used instead?

A) Split the code into multiple layers to bypass the size limit
B) Reduce the memory allocation to allow a larger package
C) Package the function as a container image, supporting up to 10 GB
D) Enable provisioned concurrency to increase the package size limit

<details>
<summary>Answer</summary>

**Correct: C**

The 250 MB unzipped limit applies to the combined size of function code and layers for .zip deployments. Container images support up to 10 GB, making them the correct choice for large dependencies.

**Why the others are wrong:**
- Layers count toward the same 250 MB unzipped ceiling, so splitting into layers does not avoid the limit
- Memory allocation is unrelated to deployment package size limits
- Provisioned concurrency affects cold starts, not packaging size limits

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** What is the maximum execution timeout that can be configured for an AWS Lambda function?

A) 3 seconds
B) 5 minutes
C) 15 minutes
D) 1 hour

<details>
<summary>Answer</summary>

**Correct: C**

Lambda supports a maximum configurable timeout of 900 seconds (15 minutes). The default value when creating a function is 3 seconds.

**Why the others are wrong:**
- 3 seconds is the default timeout, not the maximum
- 5 minutes (300 seconds) is well below the actual 900-second ceiling
- 1 hour exceeds the Lambda maximum; long-running workloads over 15 minutes should use Step Functions, ECS/Fargate, or Batch

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A company stores database credentials as Lambda environment variables and wants an additional layer of encryption beyond the default AWS managed key protection. What should they configure?

A) Enable provisioned concurrency to encrypt variables in memory
B) Specify a customer managed KMS key (CMK) for the function's environment variable encryption
C) Store the credentials in a Lambda layer instead of environment variables
D) Increase the function's memory allocation to trigger automatic encryption

<details>
<summary>Answer</summary>

**Correct: B**

Lambda encrypts environment variables at rest using an AWS managed key by default. For additional control and auditability, you can specify a customer managed CMK, and optionally use console encryption helpers to encrypt values client-side before they are stored.

**Why the others are wrong:**
- Provisioned concurrency addresses cold starts and has no role in encryption
- Layers store shared code/dependencies, not a secure mechanism for secrets, and are not encrypted the same way
- Memory allocation affects compute/CPU power, not encryption behavior

</details>

---

**Q8.** Which statement correctly distinguishes reserved concurrency from provisioned concurrency?

A) Reserved concurrency sets a dedicated maximum concurrency ceiling for a function; provisioned concurrency keeps a specified number of environments pre-initialized to reduce cold starts
B) Reserved concurrency pre-warms execution environments; provisioned concurrency sets a concurrency ceiling
C) They are two names for the same feature and are configured identically
D) Reserved concurrency only applies to asynchronous invocations, while provisioned concurrency only applies to synchronous invocations

<details>
<summary>Answer</summary>

**Correct: A**

Reserved concurrency guarantees and caps the maximum concurrent executions for a function, carving that capacity out of the account's unreserved pool. Provisioned concurrency keeps a set number of execution environments initialized and ready, directly mitigating cold-start latency.

**Why the others are wrong:**
- This reverses the two definitions
- They are distinct features with separate APIs (PutFunctionConcurrency vs PutProvisionedConcurrencyConfig) and different purposes
- Both concurrency types apply regardless of invocation model (sync, async, or event source mapping)

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

