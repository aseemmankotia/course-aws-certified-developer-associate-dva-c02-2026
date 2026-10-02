# Answer Key — Practice Test 2

**Q1. Correct: C**

This is optimistic locking: read the version, then write only if the version hasn't changed (ConditionExpression version = :expectedVersion), incrementing version on success. A ConditionalCheckFailedException tells the caller to retry with fresh data, preventing lost updates.

---

**Q2. Correct: B**

EventBridge event patterns can match deeply nested JSON fields (not just top-level message attributes), route to multiple targets, and EventBridge also natively supports scheduled (cron/rate) rules on the same bus, satisfying both requirements in one service.

---

**Q3. Correct: A**

Provisioned concurrency pre-initializes a specified number of execution environments so invocations hit warm environments immediately, eliminating cold starts. Application Auto Scaling can scale provisioned concurrency on a schedule to match the known daily spike, which reserved concurrency alone does not do.

---

**Q4. Correct: D**

FIFO queues guarantee strict ordering within a message group. Using the account ID as MessageGroupId means each account's messages are delivered/processed in order among themselves, while separate groups (different accounts) can be processed in parallel, maximizing throughput.

---

**Q5. Correct: B**

LSIs share the table's partition key (only the sort key differs), must be defined at table creation, are limited to 5 per table, and uniquely among indexes support strongly consistent reads. GSIs can use any partition/sort key combination, can be added or removed anytime, but only support eventually consistent reads and have their own provisioned/on-demand capacity.

---

**Q6. Correct: B**

S3 multipart upload lets a large object be uploaded in independent parts (5MB minimum, except the last, up to 10,000 parts, 5TB object max), each with its own presigned URL. If a part fails, only that part needs to be retried — not the whole file — dramatically improving resilience on unreliable connections, and parts can even upload in parallel.

---

**Q7. Correct: D**

TransactWriteItems supports up to 100 actions (including ConditionCheck items) that succeed or fail as a single all-or-nothing atomic unit, which is exactly what's needed to enforce the balance check and both updates together.

---

**Q8. Correct: C**

For asynchronous invocations, Lambda throttling (HTTP 429 internally) triggers the built-in async retry behavior: up to 2 additional automatic retries with delays, after which unprocessed events are sent to an on-failure destination or DLQ if configured.

---

**Q9. Correct: C**

SQS FIFO deduplication (whether content-based or via an explicit MessageDeduplicationId) is scoped to the queue as a whole for the 5-minute deduplication interval — it is not scoped per MessageGroupId, so identical-body messages sent to different groups within the interval are still deduplicated to one delivery.

---

**Q10. Correct: C**

S3 multipart upload requires every part except the last to be at least 5 MB; if the developer attempts to complete the upload with a smaller non-last part, S3 rejects it with an EntityTooSmall error during CompleteMultipartUpload.

---

**Q11. Correct: D**

API Gateway (both REST and HTTP APIs) enforces a hard maximum integration timeout of 29 seconds. Even though this particular Lambda function's own timeout is set to 15 minutes, API Gateway will return a 504 Gateway Timeout to the client after 29 seconds if the Lambda hasn't responded yet.

---

**Q12. Correct: A**

IntervalSeconds sets the first retry delay to 2 seconds, BackoffRate of 2.0 doubles the wait on each subsequent attempt (2s, 4s, 8s), and MaxAttempts of 3 caps it at three retries — matching all stated requirements exactly using valid ASL Retry field names.

---

**Q13. Correct: C**

TransactWriteItems performs an all-or-nothing atomic write across multiple items/tables and supports a ConditionExpression per item, so the inventory decrement only happens if stock is sufficient. Pairing it with a ClientRequestToken (idempotency window) or a ConditionCheck on a stored idempotency-key item ensures that a retried request does not double-decrement inventory or create duplicate orders.

---

**Q14. Correct: A**

Enabling ReportBatchItemFailures allows the function to return only the identifiers of the messages that failed. The event source mapping then deletes the successfully processed messages and makes only the failed ones visible again for retry, avoiding head-of-line blocking and duplicate processing within a FIFO message group.

---

**Q15. Correct: D**

The ExternalId condition is AWS's documented mechanism specifically for the third-party/confused deputy problem: the trust policy requires sts:ExternalId to match a customer-specific secret value, so even if a role ARN leaks or is guessed, AssumeRole fails without the correct ExternalId tied to that specific customer relationship.

---

**Q16. Correct: D**

User Pools handle authentication and issue JWTs, but they do not grant AWS credentials. An Identity Pool (Federated Identity) is required to exchange the User Pool's ID token for temporary IAM credentials via STS, allowing the app to call AWS services like S3 directly.

---

**Q17. Correct: C**

This is the standard envelope encryption pattern: GenerateDataKey returns both a plaintext data key (used locally for fast symmetric encryption of large data) and a KMS-encrypted copy of that key. The plaintext key must be discarded from memory after use, and only the encrypted data key is persisted with the ciphertext, to be decrypted later via KMS Decrypt.

---

**Q18. Correct: A**

With SSE-C, the customer supplies the encryption key with every PutObject/GetObject request over HTTPS. S3 uses that key to perform the server-side encryption/decryption but does not store the key itself — it is discarded immediately after the operation completes, satisfying the requirement that S3/KMS never retains custody of key material.

---

**Q19. Correct: A**

Secrets Manager natively supports automatic rotation with prebuilt Lambda rotation functions for RDS (single-user and multi-user rotation templates), managing the AWSCURRENT/AWSPENDING version staging so the database and secret stay in sync with zero downtime. As long as the app always retrieves the secret via GetSecretValue rather than caching it indefinitely, no redeploy is needed.

---

**Q20. Correct: A**

The trust policy (the resource-based policy attached to the role, which defines who can assume it) supports condition keys like aws:MultiFactorAuthPresent. Setting this condition to 'true' on the sts:AssumeRole action ensures STS only issues temporary credentials for the role when the calling principal's session was established with MFA.

---

**Q21. Correct: A**

The ExternalId condition on sts:AssumeRole is the AWS-recommended mechanism for third-party cross-account access. Each customer issues a unique ExternalId, and the role's trust policy requires it, so the SaaS platform cannot be tricked into assuming the wrong customer's role even if it reuses a single IAM role internally.

---

**Q22. Correct: B**

This is the standard envelope encryption pattern: GenerateDataKey returns both a plaintext data key (used once to encrypt the large object locally, then immediately discarded) and the same key encrypted under the CMK (stored with the ciphertext for later decryption via KMS Decrypt). This avoids the 4KB Encrypt API limit entirely.

---

**Q23. Correct: B**

Cognito User Pools handle authentication and issue JWTs but do not grant AWS access on their own. A Cognito Identity Pool federates that identity and exchanges the token for temporary, least-privilege IAM credentials (via GetId/GetCredentialsForIdentity under the hood), optionally applying role mapping rules based on token claims.

---

**Q24. Correct: A**

SSE-C lets S3 perform the actual encryption/decryption but requires the customer to pass the AES-256 key in the request headers on every single PUT and GET call. S3 uses the key transiently for the cryptographic operation and then discards it — it is never stored by AWS.

---

**Q25. Correct: D**

Secrets Manager's built-in rotation lifecycle always calls the four steps in this exact order: createSecret (generate new version as AWSPENDING), setSecret (apply the new credential to the actual service/database), testSecret (validate the new credential works), finishSecret (move AWSPENDING label to AWSCURRENT to complete rotation).

---

**Q26. Correct: C**

For most services including Lambda, cross-account access requires permission on BOTH sides: the resource-based policy in the resource owner's account (Account A) AND an identity-based policy in the caller's own account (Account B) explicitly allowing the action on that resource ARN. Without the identity-based grant in Account B, the call is denied by default (implicit deny) even though Account A's resource policy allows it.

---

**Q27. Correct: C**

Lifecycle hook failures (BeforeAllowTraffic, AfterAllowTraffic) cause CodeDeploy to fail the deployment and roll back, so traffic is never shifted to the new version — this is how validation gates protect production traffic.

---

**Q28. Correct: B**

CloudFormation prevents you from modifying or deleting an exported output value while any other stack has an active Fn::ImportValue reference to it — the update call fails until the dependent import is removed.

---

**Q29. Correct: D**

ECS services support three deployment controller types: ECS (rolling), CODE_DEPLOY (blue/green), and EXTERNAL. To let CodeDeploy manage traffic shifting via an AppSpec file and a load balancer, the service must explicitly use the CODE_DEPLOY controller type.

---

**Q30. Correct: D**

CodeArtifact repositories support 'external connections' to public package repositories such as npmjs, PyPI, Maven Central, and NuGet Gallery. Once configured, requests for packages not already cached are automatically fetched from the public source and cached in CodeArtifact.

---

**Q31. Correct: A**

When a CloudWatch alarm referenced in the DeploymentPreference's Alarms list enters ALARM state during a canary or linear deployment, CodeDeploy automatically triggers a rollback, repointing the alias back to the previous stable version.

---

**Q32. Correct: D**

The Immutable deployment policy launches a full set of new instances in a separate temporary Auto Scaling group alongside the existing fleet, verifies their health, then shifts traffic and terminates the old instances — providing the safest rollback and zero impact on existing capacity during validation.

---

**Q33. Correct: C**

ValidateService is the final lifecycle event in an EC2/On-Premises deployment, executed after ApplicationStart. It is specifically designed to verify that the deployment succeeded, such as by checking that the service responds correctly, before CodeDeploy marks the deployment as successful.

---

**Q34. Correct: D**

CloudFormation prevents you from deleting or modifying an exported output value as long as another stack has an active Fn::ImportValue reference to it. The update to NetworkStack will fail with an error indicating the export is in use, and the importing stack (AppStack) must be updated first to remove the dependency.

---

**Q35. Correct: B**

When a CloudWatch alarm associated with a SAM/CodeDeploy deployment enters ALARM state during a canary or linear traffic shift, CodeDeploy automatically triggers a rollback, redirecting 100% of traffic back to the original (previously stable) Lambda alias target and halting the deployment.

---

**Q36. Correct: C**

ECS supports three deployment controller types: ECS (native rolling updates), CODE_DEPLOY (enables blue/green deployments managed by AWS CodeDeploy using an appspec.yaml, two target groups, and listener rules), and EXTERNAL (for third-party deployment tools). Blue/green via CodeDeploy requires setting the deployment controller to CODE_DEPLOY.

---

**Q37. Correct: A**

Cross-account CodePipeline actions require artifacts to be encrypted with a customer-managed (not AWS-managed) KMS key, because AWS-managed keys cannot have cross-account resource policies attached. The pipeline's S3 artifact bucket policy and the KMS key policy must both grant the target account's assumed role access to decrypt and retrieve the artifacts, alongside a cross-account IAM role setup for the action.

---

**Q38. Correct: B**

Lambda Destinations for asynchronous invocations capture a full invocation record — including the original event, request/response context, and error details — and are the AWS-recommended replacement for legacy DLQs when richer failure context is needed for reprocessing.

---

**Q39. Correct: A**

Each DynamoDB physical partition has a hard throughput ceiling regardless of table-level provisioning. A low-cardinality key concentrates traffic on a few partitions ('hot partition'). Redistributing writes via a high-cardinality or sharded key spreads traffic across more partitions and resolves throttling.

---

**Q40. Correct: B**

Initializing the connection in the global scope allows it to be reused across invocations that share a warm execution environment, and RDS Proxy pools and multiplexes connections so that Lambda's rapid concurrency scaling doesn't exhaust the database's native connection limit.

---

**Q41. Correct: D**

Lambda proxy integration requires the function to return a specific JSON structure with an integer statusCode and a body that is a JSON-stringified string. If any code path returns a malformed structure (missing statusCode, non-string body, or an unhandled exception), API Gateway cannot parse it and returns 502 'Malformed Lambda proxy response.'

---

**Q42. Correct: B**

DAX is a DynamoDB-API-compatible, fully managed in-memory cache purpose-built for DynamoDB. It provides microsecond read latency and requires only swapping the DynamoDB client for the DAX client, with minimal application code changes.

---

**Q43. Correct: B**

Annotations are indexed key-value pairs (string, number, or boolean) that can be used to filter and group traces in the X-Ray console or via GetTraceSummaries. Metadata stores arbitrary, larger data objects that are recorded with the trace but are not indexed or searchable — ideal for large debug payloads.

---

**Q44. Correct: A**

Every DynamoDB partition (regardless of capacity mode) has a hard per-partition throughput ceiling (roughly 3,000 RCU / 1,000 WCU). Using orderDate as the partition key means every order placed on the same day lands on the same physical partition, creating a hot partition that throttles once that ceiling is hit — even though the table's aggregate on-demand capacity is nowhere near its limit. The only durable fix is a partition key redesign (higher cardinality or write sharding) that distributes traffic across many partitions.

---

**Q45. Correct: C**

X-Ray annotations are indexed key-value pairs (string, number, or boolean only) that can be used in X-Ray console/API filter expressions — ideal for a simple searchable field like customerId. Metadata is unindexed and can hold arbitrary, larger objects like a full JSON payload; it's stored with the trace for debugging but is never searchable, which is exactly the separation needed here.

---

