# Answer Key — Practice Test 1

**Q1. Correct: B**

Reserved concurrency sets a hard cap on the maximum number of concurrent executions for that specific function, throttling any invocations beyond the limit so the database never receives more than 10 simultaneous connections.

---

**Q2. Correct: A**

A ConditionExpression tied to a version number (e.g., version = :expectedVersion) makes the write atomic and conditional: DynamoDB rejects the update with a ConditionalCheckFailedException if another process already changed the item, which is the standard optimistic locking pattern tested on the exam.

---

**Q3. Correct: C**

A presigned URL is only valid as long as both conditions hold: the ExpiresIn window has not elapsed AND the credentials used to sign it are still valid. Since the assumed role's temporary credentials expired after 1 hour, the request fails with an authentication error at 2 hours despite the 7-day ExpiresIn setting.

---

**Q4. Correct: C**

FIFO queues guarantee ordering only within a message group. Setting MessageGroupId to the customer ID ensures each customer's orders are processed strictly in order, while different customers' message groups can be consumed in parallel, maximizing throughput.

---

**Q5. Correct: A**

Step Functions is purpose-built for orchestrating multi-step workflows with built-in support for sequencing, conditional branching (Choice states), long wait periods (Wait states, including waiting for a human approval callback via waitForTaskToken), and configurable retry/catch logic per state — exactly matching every requirement.

---

**Q6. Correct: C**

Without jitter, many clients backing off using the same exponential formula will retry at nearly the same moments, producing synchronized bursts of load ('thundering herd') that can re-trigger throttling. Adding randomized jitter spreads retries out over time, smoothing load on the service.

---

**Q7. Correct: C**

Optimistic locking with a version attribute and a ConditionExpression makes the UpdateItem call fail with a ConditionalCheckFailedException if another process already changed the item, letting the caller detect the conflict and retry instead of blindly overwriting data.

---

**Q8. Correct: B**

FIFO queues guarantee exactly-once processing and strict ordering, and setting MessageGroupId per customer ensures ordering is preserved within each customer's message group while allowing parallelism across different customers.

---

**Q9. Correct: C**

EventBridge is purpose-built for ingesting and routing events from SaaS partners (via partner event sources), supports schema registry/discovery, and can route to multiple targets across accounts using rules based on source and detail-type — all without custom code.

---

**Q10. Correct: D**

Reserved concurrency sets both a guaranteed and a maximum number of concurrent executions for a function, so setting it to 10 caps concurrent invocations at 10, protecting the downstream RDS instance from connection exhaustion.

---

**Q11. Correct: B**

A presigned POST allows the server to embed a policy document with conditions such as content-type restrictions and content-length-range, enforcing the file type and size limits while letting the browser upload directly to S3 without backend involvement.

---

**Q12. Correct: C**

Adding a randomized jitter value to exponential backoff spreads out retry attempts from many clients over time, preventing them from all retrying simultaneously and overwhelming the service again — a well-documented AWS SDK best practice.

---

**Q13. Correct: C**

A conditional PutItem using attribute_not_exists(paymentId) is atomic in DynamoDB, so only the first invocation for a given paymentId succeeds; any duplicate invocation gets a ConditionalCheckFailedException and can skip reprocessing. This guarantees idempotency regardless of how many times the function is invoked for the same message.

---

**Q14. Correct: D**

AWS Step Functions Standard Workflows natively support sequential and branching state machines, Wait states for long external callbacks (including task tokens that can pause for up to a year), built-in retry and catch configuration with exponential backoff, and a visual console showing the execution history of every state transition, which matches every requirement in the scenario.

---

**Q15. Correct: B**

A cross-account role with a trust policy requiring an external ID and assumed via STS AssumeRole prevents the confused deputy problem and issues short-lived, auditable temporary credentials instead of long-term keys.

---

**Q16. Correct: B**

This is the standard Cognito pattern: the User Pool authenticates the user and issues JWT tokens, and those tokens are federated into an Identity Pool, which uses STS to vend temporary IAM credentials for direct S3 access.

---

**Q17. Correct: D**

A bucket policy Deny statement with a condition checking that the s3:x-amz-server-side-encryption header is not 'aws:kms' actively rejects any upload request that omits or misconfigures KMS encryption, enforcing the requirement at the API level.

---

**Q18. Correct: D**

AWS Secrets Manager natively supports scheduled automatic rotation for RDS credentials using AWS-provided Lambda rotation templates, requiring no custom rotation logic and no application downtime since the rotation is transparent.

---

**Q19. Correct: A**

Envelope encryption works by having KMS generate a unique data key returned in two forms — plaintext and CMK-encrypted; the application uses the plaintext key locally for fast symmetric encryption of large data, then discards it and stores only the encrypted copy for future decryption via KMS Decrypt.

---

**Q20. Correct: B**

S3-to-Lambda invocation requires a resource-based policy on the Lambda function (added via lambda:AddPermission, or automatically by SAM/CloudFormation event source definitions) that explicitly authorizes the S3 service principal, scoped to the source bucket, to invoke the function.

---

**Q21. Correct: D**

Cross-account IAM roles with a trust policy plus sts:AssumeRole is the standard, secure pattern for granting temporary, scoped, auditable access across accounts without distributing long-term credentials.

---

**Q22. Correct: D**

Cognito Identity Pools (federated identities) are purpose-built to exchange tokens from external identity providers like Google or Facebook for temporary, scoped AWS credentials via STS, without the developer having to manage federation logic.

---

**Q23. Correct: A**

Envelope encryption uses GenerateDataKey to obtain a plaintext key (for fast local encryption of large data) and its KMS-encrypted counterpart (stored for future decryption); the plaintext key must never persist after use.

---

**Q24. Correct: B**

SSE-S3 uses AES-256 keys that Amazon S3 creates, manages, and rotates automatically at no additional cost, with no KMS API calls or key policy configuration required.

---

**Q25. Correct: C**

AWS Secrets Manager natively supports automatic secret rotation with built-in Lambda rotation function templates for RDS, Redshift, and DocumentDB, requiring no custom scheduling infrastructure.

---

**Q26. Correct: D**

Identity-based policies attach to IAM principals (users, groups, roles) and implicitly apply to that principal, while resource-based policies attach to the resource itself and include a Principal element specifying who is granted access, which is commonly used for cross-account access.

---

**Q27. Correct: B**

Linear10PercentEvery10Minutes shifts traffic in equal 10% increments every 10 minutes until reaching 100%, matching the described gradual linear rollout with alarm monitoring time between increments.

---

**Q28. Correct: B**

Fn::ImportValue retrieves values that have been exported by another stack using the Export field in Outputs, enabling cross-stack references such as sharing a VPC ID between a network stack and an application stack.

---

**Q29. Correct: A**

The SAM template's Globals section lets you define common properties such as MemorySize, Timeout, and Environment variables once, and SAM automatically applies them to every AWS::Serverless::Function and AWS::Serverless::Api resource in the template, unless overridden individually.

---

**Q30. Correct: D**

Granting the CodeBuild service role IAM permissions for codeartifact:GetAuthorizationToken (and related actions) and running 'aws codeartifact login --tool npm' in the install phase lets CodeBuild dynamically obtain a short-lived authorization token to authenticate npm without hardcoding credentials.

---

**Q31. Correct: A**

AWS CodeDeploy supports the ECS compute platform natively, provisioning a new task set alongside the old one behind an Application Load Balancer, shifting traffic (linear, canary, or all-at-once), and automatically rolling back based on configured CloudWatch alarms — this is the standard blue/green mechanism for ECS/Fargate.

---

**Q32. Correct: A**

The Immutable deployment policy launches an entirely new set of instances (in a temporary Auto Scaling group) alongside the full original fleet, and only shifts traffic once the new instances pass health checks; if they fail, Elastic Beanstalk simply terminates the new instances, leaving the original fleet completely untouched and minimizing downtime risk.

---

**Q33. Correct: D**

For Lambda deployments, CodeDeploy supports the BeforeAllowTraffic and AfterAllowTraffic lifecycle hooks. BeforeAllowTraffic runs before any traffic is routed to the new version, making it the correct place to run validation logic. Pairing it with CloudWatch alarms on the deployment group enables automatic rollback if the validation Lambda reports failure.

---

**Q34. Correct: B**

A CloudFormation change set generates a summary of proposed changes, including resource additions, modifications, and replacements, without actually applying them, allowing the team to review impact before execution.

---

**Q35. Correct: A**

The Globals section in a SAM template lets developers define common properties, such as Timeout, MemorySize, and Runtime, once, and those settings apply to all applicable AWS::Serverless::Function (and AWS::Serverless::Api) resources unless overridden at the individual resource level.

---

**Q36. Correct: D**

Setting the ECS service deployment controller to CODE_DEPLOY integrates the service with AWS CodeDeploy, enabling true blue/green deployments with a new task set behind a test/production listener on an ALB, and CloudWatch alarms can trigger automatic rollback during the traffic shift.

---

**Q37. Correct: A**

AWS CodeArtifact is a fully managed artifact repository service built specifically for software packages, supporting package managers like Maven and npm, with built-in dependency resolution and the ability to share repositories across accounts using resource policies.

---

**Q38. Correct: D**

Annotations are indexed key-value pairs on a segment or subsegment that can be used to filter and group traces in the X-Ray console or via GetTraceSummaries; this is the only mechanism designed for searchable custom fields.

---

**Q39. Correct: B**

Using order_date as the partition key means every write for a given day targets the same physical partition. Each partition has hard throughput limits (1,000 WCU / 3,000 RCU), so throttling occurs there even if the table's total provisioned throughput is unused elsewhere — the classic hot partition problem.

---

**Q40. Correct: A**

Lambda destinations for asynchronous invocations capture a full invocation record — including the original event, the response payload, and error details — and route it to SQS, SNS, another Lambda function, or EventBridge, all without any code changes.

---

**Q41. Correct: C**

In Lambda proxy integration mode, API Gateway requires the function to return a specific JSON structure with statusCode, headers, and body fields. If the returned object doesn't conform, API Gateway cannot parse it and returns a 502 even though the Lambda invocation itself succeeded.

---

**Q42. Correct: A**

DAX is a DynamoDB-compatible, in-memory cache that sits in front of DynamoDB and is accessed through the DAX client, which is API-compatible with the DynamoDB SDK. This means applications can adopt it with minimal code changes while achieving microsecond read latency.

---

**Q43. Correct: C**

CloudWatch Logs Insights uses its own query language: filter to select matching log events, and stats ... by bin(interval) to aggregate results into time buckets. This is the correct, valid syntax.

---

**Q44. Correct: B**

DynamoDB enforces a per-partition throughput limit (3,000 RCU / 1,000 WCU) regardless of capacity mode or overall table-level throughput. With only 3 distinct partition key values, all traffic concentrates on 3 physical partitions, causing throttling even when the table's aggregate consumed capacity is low. Using a high-cardinality key like session_id spreads writes and reads evenly across many partitions, eliminating the hot partition.

---

**Q45. Correct: C**

Annotations are indexed by X-Ray and are searchable using filter expressions in the console, but they only support simple scalar key-value pairs (string, number, boolean). Metadata is not indexed or searchable but can store arbitrarily large and complex objects, making it ideal for the full JSON request payload. customer_id belongs in annotations since it must be filterable; the JSON payload belongs in metadata since it is large and does not need to be searched.

---

