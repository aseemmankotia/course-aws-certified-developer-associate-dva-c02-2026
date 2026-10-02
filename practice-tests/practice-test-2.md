# Practice Test 2: AWS Certified Developer Associate DVA-C02 Prep

- **Questions:** 45
- **Time limit:** 90 minutes
- **Passing score:** 70%%

> Answers are in `answer-key-2.md`. Track results in `score-tracker.md`.

---

**Q1.** A DynamoDB table stores inventory counts. Two Lambda functions running concurrently both read an item's 'stock' attribute and 'version' attribute, then attempt to decrement stock and increment version. Without any coordination, one update can silently overwrite the other, causing incorrect stock counts. Which approach BEST prevents this lost-update problem?

A) Enable DynamoDB Streams and reconcile conflicting updates in a downstream Lambda after they occur
B) Wrap both operations in a TransactWriteItems call with no ConditionCheck items, relying on transactions alone to serialize access
C) Use UpdateItem with a ConditionExpression that checks the version attribute still equals the value read, and increment the version atomically on success
D) Add DynamoDB Accelerator (DAX) in front of the table to serialize concurrent writes through the cache layer

---

**Q2.** An order-processing platform publishes order events containing deeply nested JSON with fields like order.items[].category and order.customer.tier. Different consuming services need to react only when specific nested field values match certain patterns, and some rules must also fire on a recurring schedule independent of incoming events. Which service should be used to route and filter these events?

A) Amazon SQS standard queues with per-consumer message attribute filtering
B) Amazon EventBridge with event pattern rules matching nested JSON content and scheduled rules
C) Amazon SNS topics with subscription filter policies based on top-level message attributes
D) AWS Step Functions Standard Workflows with Choice states evaluating the event payload

---

**Q3.** A Lambda function backing a mobile app has reserved concurrency set to 50 to protect a downstream RDS database. During a marketing push, traffic spikes sharply at a known time each morning and users report multi-second delays during the first minute of the spike, even though total concurrent invocations stay under 50. What should be configured to eliminate this cold-start latency for the predictable spike?

A) Configure provisioned concurrency for the function (or a published version/alias) scaled up ahead of the known traffic spike via Application Auto Scaling
B) Increase reserved concurrency from 50 to 200 so more warm environments are available
C) Enable AWS X-Ray active tracing on the function to reduce initialization overhead
D) Increase the function's memory allocation to the maximum 10,240 MB to speed up initialization

---

**Q4.** A banking application must process withdrawal and deposit transactions for each customer account strictly in the order they occurred, but transactions for different customer accounts can be processed in any order relative to each other, and the system must handle thousands of accounts with high overall throughput. Which SQS configuration meets this requirement?

A) A standard SQS queue, since standard queues process messages roughly in the order sent for the majority of cases
B) A FIFO queue with a single fixed MessageGroupId for all transactions to guarantee global ordering
C) An SNS topic fanning out to one SQS queue per customer account, created dynamically as new customers register
D) A FIFO queue using the customer account ID as the MessageGroupId, allowing parallel processing across groups while preserving order within each group

---

**Q5.** A developer is deciding between a DynamoDB Local Secondary Index (LSI) and a Global Secondary Index (GSI) for a new access pattern. Which statement correctly distinguishes the two?

A) An LSI can be added to a table at any time after creation, while a GSI must be defined when the table is created
B) An LSI must share the base table's partition key and can be queried with strong consistency, while a GSI can use a completely different partition key but only supports eventually consistent reads
C) A GSI supports strongly consistent reads by default, while an LSI only supports eventually consistent reads
D) Both LSIs and GSIs consume the base table's provisioned throughput and cannot have independent read/write capacity settings

---

**Q6.** A web application allows users to upload video files up to 2GB directly to S3 from the browser using presigned URLs. On unreliable mobile connections, uploads frequently fail partway through, forcing users to restart the entire upload from byte zero, causing high abandonment. Which change to the upload approach BEST solves this?

A) Generate a single presigned PUT URL with a longer expiration time so the browser has more time to retry the whole upload
B) Use S3 multipart upload: split the file into parts (minimum 5MB each, except the last part), generate a presigned URL per part, and retry only the failed parts before completing the upload
C) Switch to S3 Transfer Acceleration for the presigned URL to speed up the single PUT request and reduce failure likelihood
D) Store the file temporarily in Amazon EFS during upload, then copy it to S3 once the transfer is fully validated as complete

---

**Q7.** A DynamoDB table stores bank account balances with AccountId as the partition key. An application must transfer $50 from Account A to Account B, but only if Account A's balance is at least $50. Both the balance check and the two balance updates must succeed or fail together as a single atomic unit. Which DynamoDB operation should the developer use?

A) BatchWriteItem with a ConditionExpression applied to each PutItem request in the batch
B) PutItem on Account A followed sequentially by PutItem on Account B, each with its own ConditionExpression
C) UpdateItem on Account A with ReturnValues set to ALL_OLD, then manually reversing the update in application code if Account B's update fails
D) TransactWriteItems containing a ConditionCheck on Account A's balance plus two Update actions to debit and credit the accounts

---

**Q8.** A Lambda function is invoked asynchronously by S3 event notifications. Reserved concurrency for the function is set to 10, and no provisioned concurrency is configured. During a traffic spike, 50 concurrent invocation attempts occur simultaneously. What happens to the invocation requests beyond the reserved concurrency limit of 10?

A) They fail immediately with no retry because S3 event notifications are fire-and-forget with no built-in retry mechanism
B) API Gateway returns a 503 error back to the S3 event source, which triggers S3 to automatically resend the notification
C) Lambda throttles the excess asynchronous invocations and automatically retries them up to two additional times with delays; if all retries are exhausted, the events are sent to the configured on-failure destination or DLQ
D) The account's unreserved concurrency pool automatically absorbs the excess invocations, bypassing the function's reserved concurrency limit

---

**Q9.** An SQS FIFO queue has content-based deduplication enabled. Two separate SendMessage calls are made within the same 5-minute deduplication interval: both messages have identical message bodies, but they are sent with different MessageGroupId values. What is the result?

A) Both messages are delivered because deduplication is scoped per MessageGroupId, so identical bodies in different groups are treated as distinct
B) Both messages are delivered because content-based deduplication only applies to standard queues, not FIFO queues
C) Only one copy of the message is delivered, because SQS FIFO content-based deduplication computes the body hash and applies deduplication at the queue level, not scoped per message group
D) Only the second message is delivered, and it silently overwrites the first message already sitting in the queue

---

**Q10.** A developer is implementing a multipart upload to Amazon S3 for a large file using the AWS SDK, uploading parts out of order across multiple threads. Aside from the last part, what is the minimum allowed size for an individual part, and what happens if a non-last part is uploaded below that size?

A) 100 MB minimum; CompleteMultipartUpload returns an InvalidPartOrder error for any undersized part
B) 1 MB minimum; the SDK automatically pads undersized parts to meet the requirement
C) 5 MB minimum; CompleteMultipartUpload will fail with an EntityTooSmall error if any part except the last is below 5 MB
D) There is no minimum part size restriction; S3 accepts parts of any size and reassembles them regardless of order

---

**Q11.** A REST API in API Gateway uses Lambda proxy integration to invoke a function configured with a 15-minute timeout to run a long batch job synchronously in response to a client's HTTPS request. What is the actual maximum time the client will wait before receiving a timeout response, regardless of the Lambda function's configured timeout?

A) 15 minutes, because API Gateway proxy integration always waits for the full Lambda execution to complete before timing out
B) 60 seconds, which is the default and can be extended to match the Lambda function's configured timeout via a console setting
C) 300 seconds, which is the default integration timeout applied uniformly across REST and HTTP APIs
D) 29 seconds, because API Gateway enforces a maximum integration timeout that is independent of and shorter than the Lambda function's own timeout setting

---

**Q12.** In an AWS Step Functions state machine defined in Amazon States Language, a Task state invokes a Lambda function that occasionally throws Lambda.ServiceException due to throttling. The requirement is: retry after 2 seconds initially, doubling the wait time on each subsequent attempt, for up to 3 retry attempts. Which Retry block correctly implements this?

A) {"ErrorEquals": ["Lambda.ServiceException"], "IntervalSeconds": 2, "MaxAttempts": 3, "BackoffRate": 2.0}
B) {"ErrorEquals": ["Lambda.ServiceException"], "IntervalSeconds": 2, "MaxAttempts": 3, "BackoffRate": 1.0}
C) {"ErrorEquals": ["Lambda.ServiceException"], "IntervalSeconds": 1, "MaxAttempts": 2, "BackoffRate": 2.0}
D) {"ErrorEquals": ["Lambda.ServiceException"], "RetryCount": 3, "IntervalSeconds": 2, "BackoffRate": 2.0}

---

**Q13.** An order-processing service must atomically decrement an inventory count and create an order record only if sufficient inventory exists, and the operation must be safely retryable if the client resends the same request due to a network timeout. Which approach BEST meets these requirements using DynamoDB?

A) Perform a PutItem for the order followed by an UpdateItem for inventory in separate calls, wrapped in a try/catch that manually reverses the order write if the inventory update fails
B) Use BatchWriteItem to write the order item and update the inventory item in a single batch request so both changes commit together
C) Use TransactWriteItems with a ConditionExpression on the inventory item to prevent overselling, combined with a ConditionCheck against a stored idempotency key (or a ClientRequestToken) to make retries safe
D) Enable DynamoDB Streams on the inventory table and have a downstream Lambda function reverse the inventory decrement if the order record is missing

---

**Q14.** A Lambda function is triggered by an Amazon SQS FIFO queue with a batch size of 10. Occasionally, 2 of the 10 messages in a batch fail processing due to a downstream validation error, while the other 8 succeed. The team needs the 2 failed messages to be retried without reprocessing the 8 successful ones, and without stalling the rest of the message group. Which configuration change accomplishes this?

A) Enable ReportBatchItemFailures on the event source mapping's FunctionResponseTypes and have the function return a batchItemFailures list containing the itemIdentifier of only the 2 failed messages
B) Set the SQS queue's visibility timeout to 0 seconds so any message that isn't explicitly deleted immediately becomes visible again
C) Configure reserved concurrency of 1 on the Lambda function so messages are processed strictly one at a time
D) Wrap the processing logic in a try/catch that logs errors internally and always returns a successful response to the event source mapping

---

**Q15.** A SaaS vendor's application assumes an IAM role in each customer's AWS account by calling sts:AssumeRole against a role ARN the customer creates. Security review flags a potential 'confused deputy' risk: if the role ARN or trust relationship is guessed or misconfigured, the vendor's application could be tricked into assuming a role belonging to a different customer than intended. What should the vendor implement to mitigate this risk?

A) Require MFA on every AssumeRole call made by the vendor's backend service
B) Reduce the assumed role's session duration to the minimum allowed value of 15 minutes
C) Replace the cross-account role with an S3 bucket resource policy granting the vendor's account direct access
D) Have each customer include a unique ExternalId in the role's trust policy, and require the vendor's AssumeRole calls to pass that same ExternalId

---

**Q16.** A mobile app authenticates users against an Amazon Cognito User Pool and receives an ID token and access token (JWTs) after login. The app must now call S3 PutObject directly from the device using temporary AWS credentials scoped to that user via IAM. Which additional Cognito component is required to obtain those temporary AWS credentials?

A) Increase the User Pool app client's access token expiration time so it can be used directly as AWS credentials
B) Enable a custom authentication challenge Lambda trigger on the User Pool
C) Attach an IAM policy directly to the User Pool app client
D) A Cognito Identity Pool configured to accept the User Pool as an authentication provider and vend temporary credentials via AssumeRoleWithWebIdentity

---

**Q17.** A developer needs to encrypt a 500 MB file client-side using AWS KMS before uploading it to S3. The KMS Encrypt API has a data size limit that makes it unsuitable for encrypting the file directly. What is the correct envelope encryption workflow using the AWS Encryption SDK or SDK KMS calls?

A) Call KMS Encrypt directly on the full file since KMS supports payloads up to 6 MB
B) Store the plaintext data key returned by GenerateDataKey in Secrets Manager so it can be reused for future files
C) Call GenerateDataKey to obtain a plaintext data key and its encrypted copy, encrypt the file locally with the plaintext key, discard the plaintext key, then store the encrypted data key alongside the ciphertext
D) Call Decrypt first to derive a reusable data key, then use that key to encrypt the file

---

**Q18.** A financial services company's compliance policy states that AWS must never generate, store, or manage the master encryption key material used to protect a specific S3 bucket's objects — the customer's own key management system must retain full custody of the key at all times, while S3 still performs the actual server-side encrypt/decrypt operation on upload and download. Which S3 encryption option satisfies this requirement?

A) SSE-C (server-side encryption with customer-provided keys)
B) SSE-S3
C) SSE-KMS using a customer managed CMK
D) Client-side encryption using an AWS KMS customer managed CMK

---

**Q19.** An application stores an RDS MySQL database credential and requires it to rotate automatically every 30 days with zero application downtime, without any code redeployment when the password changes. Which approach correctly satisfies this requirement?

A) Store the credential in AWS Secrets Manager, enable automatic rotation using the AWS-provided RDS rotation Lambda function, and have the application call GetSecretValue at runtime instead of hardcoding the password
B) Store the credential in Systems Manager Parameter Store as a SecureString and configure a TTL policy that forces expiration after 30 days
C) Store the credential in the Lambda function's environment variables encrypted with a KMS key, and manually update the environment variable every 30 days
D) Store the credential in Parameter Store and set up a CloudWatch Events rule that emails an administrator to manually rotate it every 30 days

---

**Q20.** A company wants to require that a highly privileged 'ProductionAdmin' IAM role can only be assumed by users who have authenticated with multi-factor authentication (MFA) in their current session. Which trust policy configuration correctly enforces this?

A) Add a Condition block to the role's trust policy requiring aws:MultiFactorAuthPresent to equal true on the sts:AssumeRole action
B) Add a permissions boundary to the role that denies all actions unless MFA is present
C) Set the role's maximum session duration to 1 hour to force frequent re-authentication
D) Attach an SCP at the organization root that denies AssumeRole for all roles named 'ProductionAdmin'

---

**Q21.** A third-party SaaS platform needs to assume an IAM role in each customer's AWS account to pull billing data via API calls. To prevent the confused deputy problem where another SaaS customer could trick the platform into accessing the wrong account, which trust policy configuration should the customer configure when creating the role?

A) Add a Condition block requiring sts:ExternalId to match a unique value issued by the customer to the SaaS platform
B) Require MFA on every AssumeRole call from the SaaS platform's IAM user
C) Restrict the trust policy to the SaaS platform's account ID only, with no additional conditions
D) Grant the SaaS platform an IAM access key scoped only to the billing API actions

---

**Q22.** A developer needs to encrypt a 12GB backup file client-side before uploading it to S3, using a CMK stored in AWS KMS. The KMS Encrypt API has a 4KB payload limit. Which approach correctly implements envelope encryption for this file?

A) Call the KMS Decrypt API to generate a reusable encryption key, then encrypt the file locally with that key
B) Call GenerateDataKey to obtain a plaintext data key and its KMS-encrypted counterpart, encrypt the file locally with the plaintext key, then discard the plaintext key and store only the encrypted data key alongside the ciphertext
C) Split the file into 4KB chunks and call the KMS Encrypt API on each chunk individually
D) Store the CMK's plaintext key material locally and reuse it to encrypt every file without further calls to KMS

---

**Q23.** A mobile app authenticates users against a Cognito User Pool and receives ID and access JWTs. The app then needs temporary AWS credentials to write directly to a DynamoDB table using the AWS SDK. Which flow correctly obtains those credentials while enforcing user-level authorization?

A) Embed a static IAM access key pair in the mobile app configured with DynamoDB read/write permissions
B) Configure a Cognito Identity Pool with the User Pool as an authentication provider, and exchange the ID token for temporary IAM credentials scoped by role mapping
C) Pass the User Pool access token directly as the AWS SigV4 signing key for DynamoDB requests
D) Call sts:GetSessionToken using the User Pool's client secret as the MFA code

---

**Q24.** A financial services company must upload objects to S3 such that S3 performs the encryption/decryption, but AWS must never persist or have access to the encryption key material at rest — the client must supply the exact same key with every GET and PUT request. Which S3 encryption option meets this requirement?

A) SSE-C, supplying the customer-provided key in the request headers on every call
B) SSE-S3 with S3-managed keys
C) SSE-KMS with a customer-managed CMK
D) Client-side encryption using the S3 Encryption Client before upload

---

**Q25.** A developer is implementing a custom Lambda rotation function for a Secrets Manager secret. AWS Secrets Manager invokes the function multiple times during a single rotation cycle. What is the correct order of the four rotation steps executed by Secrets Manager?

A) testSecret, createSecret, setSecret, finishSecret
B) createSecret, testSecret, setSecret, finishSecret
C) setSecret, testSecret, createSecret, finishSecret
D) createSecret, setSecret, testSecret, finishSecret

---

**Q26.** Account A owns a Lambda function and attaches a resource-based policy granting lambda:InvokeFunction to Account B. A developer in Account B has no identity-based IAM policy statement allowing lambda:InvokeFunction on that function's ARN. When the Account B developer attempts to invoke the function, what happens?

A) The call succeeds, because a resource-based policy alone is always sufficient to grant cross-account access regardless of the caller's own IAM policies
B) The call fails only if Account A also has an explicit Deny statement targeting Account B
C) The call fails, because for cross-account access to most AWS resources (including Lambda), the calling principal also needs an identity-based policy in their own account explicitly allowing the action
D) The call succeeds, but only if the developer in Account B is using the root user credentials

---

**Q27.** A developer deploys an update to a Lambda function using AWS CodeDeploy with an AppSpec file that defines a BeforeAllowTraffic hook. The hook function runs integration tests against the new Lambda version's alias before traffic shifts. The hook function returns a failure status. What happens to the deployment?

A) CodeDeploy logs a warning but proceeds to shift traffic to the new version anyway
B) The AfterAllowTraffic hook is triggered instead to compensate for the failure
C) The deployment fails and rolls back; traffic remains pointed at the original Lambda version
D) CodeDeploy retries the BeforeAllowTraffic hook indefinitely until it succeeds

---

**Q28.** A CloudFormation stack named NetworkStack exports a subnet ID value using the Export field in its Outputs section. Another stack, AppStack, imports this value with Fn::ImportValue. A developer attempts to update NetworkStack to change the exported subnet ID value. What is the result?

A) The update succeeds and CloudFormation automatically updates AppStack to use the new value
B) The update fails with an error because the export is still in use by another stack
C) The update succeeds, but AppStack silently continues using the old subnet ID until manually refreshed
D) CloudFormation creates a versioned export so both the old and new values remain available

---

**Q29.** A team wants AWS CodeDeploy to manage blue/green deployments for an Amazon ECS service, shifting traffic between task sets behind an Application Load Balancer. Which ECS service configuration setting is required to enable this?

A) The service's deployment controller type must be set to ECS
B) The service must use the ROLLING update type with minimum healthy percent set to 200
C) The service must be configured with the EXTERNAL deployment controller type
D) The service's deployment controller type must be set to CODE_DEPLOY

---

**Q30.** A company wants to standardize on AWS CodeArtifact as the single package source for its build systems, while still allowing developers to pull public npm packages through that same CodeArtifact repository without manually re-uploading each dependency. Which configuration meets this requirement?

A) Enable public internet access directly on the CodeArtifact domain settings
B) Have developers configure npm to pull from the public registry, then run a script to manually upload each package to CodeArtifact
C) Create an upstream repository that references another private CodeArtifact repository only, bypassing npm entirely
D) Create a CodeArtifact repository and configure an external connection to the public npm registry

---

**Q31.** A SAM template defines AutoPublishAlias: live along with a DeploymentPreference of Type: Canary10Percent5Minutes and an Alarms list pointing to a CloudWatch alarm on function errors. During deployment, 10% of traffic is shifted to the new version. At minute 3 of the 5-minute canary window, the CloudWatch alarm enters ALARM state. What happens next?

A) CodeDeploy automatically rolls back the alias, shifting traffic back to the previous Lambda version
B) CodeDeploy waits until minute 5, then shifts the remaining 90% of traffic as originally scheduled
C) The deployment pauses indefinitely until an operator manually approves or rejects it
D) The alarm is ignored because the canary phase has already started

---

**Q32.** An Elastic Beanstalk environment needs a deployment policy that launches an entirely new, fully-sized set of instances in a temporary Auto Scaling group, runs health checks against them, shifts all traffic to the new instances only after they pass, and automatically terminates the old instances afterward — minimizing any risk of degrading current production capacity during the deployment. Which Elastic Beanstalk deployment policy satisfies this?

A) All at once
B) Rolling with additional batch
C) Traffic splitting
D) Immutable

---

**Q33.** A developer is using AWS CodeDeploy to perform an in-place deployment to a fleet of EC2 instances. The AppSpec file defines lifecycle event hooks. In which hook should the developer run a script that checks HTTP 200 responses from the newly deployed application before it is considered a successful deployment?

A) BeforeInstall
B) AfterInstall
C) ValidateService
D) ApplicationStart

---

**Q34.** A developer manages two CloudFormation stacks: NetworkStack, which exports a VPC ID via an Outputs section, and AppStack, which imports that VPC ID using Fn::ImportValue. The developer attempts to update NetworkStack to remove the exported output entirely. What happens?

A) The update succeeds immediately, and AppStack automatically receives an empty value on its next deployment
B) The update triggers a change set that must be manually approved by the owner of AppStack before proceeding
C) The update succeeds but AppStack is automatically deleted along with the removed export
D) The update fails because a stack's exported output cannot be removed or modified while another stack has an active import of it

---

**Q35.** A developer configures an AWS SAM template with AutoPublishAlias set and a DeploymentPreference Type of Canary10Percent10Minutes, including a CloudWatch alarm ARN in the Alarms property. During the deployment, the specified alarm enters ALARM state 3 minutes after 10% of traffic has shifted to the new Lambda version. What does CodeDeploy do?

A) It continues the deployment on schedule since the 10-minute canary window has not yet elapsed
B) It automatically rolls back all traffic to the previous Lambda version alias target and stops the deployment
C) It immediately shifts 100% of traffic to the new version to minimize exposure to the old, believed-faulty version
D) It pauses the deployment indefinitely and waits for a developer to manually resolve the alarm

---

**Q36.** A developer wants to perform a blue/green deployment for an Amazon ECS service running on AWS Fargate, using AWS CodeDeploy to gradually shift traffic between two target groups behind an Application Load Balancer. Which ECS service deployment controller type must be configured for this to work?

A) ECS
B) EXTERNAL
C) CODE_DEPLOY
D) BLUE_GREEN

---

**Q37.** A company runs an AWS CodePipeline where the CodeBuild build stage executes in Account A, and the CodeDeploy deployment stage targets EC2 instances in Account B. Which configuration is required for this cross-account pipeline to function correctly?

A) Configure the pipeline's artifact store to use a customer-managed AWS KMS key, and grant Account B access via both the S3 bucket policy and the KMS key policy
B) Attach an IAM role in Account A with AdministratorAccess permissions scoped to Account B's resources
C) Enable S3 Cross-Region Replication between the artifact buckets in Account A and Account B
D) Attach AWS Organizations Service Control Policies granting Account B implicit access to Account A's pipeline resources

---

**Q38.** A Lambda function is invoked asynchronously by S3 event notifications when objects are uploaded to a bucket. Occasionally the function fails after exhausting all automatic retries, and the team needs to capture the failed event along with the invocation's response payload and error details for automated reprocessing. Which solution BEST meets this requirement?

A) Configure a Dead Letter Queue (DLQ) on the Lambda function pointing to an SQS queue
B) Configure an on-failure Lambda Destination pointing to an SQS queue
C) Enable AWS X-Ray tracing on the function to capture failure details
D) Configure the S3 event notification to retry indefinitely until the function succeeds

---

**Q39.** A DynamoDB table uses a partition key named OrderStatus that has only four possible values (NEW, PROCESSING, SHIPPED, COMPLETE). The table is experiencing ProvisionedThroughputExceededException errors even though the table's aggregate consumed capacity is well below its total provisioned capacity. Which change would BEST resolve this issue?

A) Redesign the partition key to include a high-cardinality attribute, such as appending a random suffix or incorporating the OrderID
B) Switch the table's billing mode to On-Demand capacity mode
C) Enable DynamoDB Accelerator (DAX) in front of the table
D) Increase the table's provisioned WriteCapacityUnits (WCU) using auto scaling

---

**Q40.** A Lambda function connects to an Amazon RDS MySQL database, creating a new JDBC connection inside the handler method on every invocation. Under high concurrency, the database reports 'too many connections' errors and Lambda invocations show increased latency. Which combination of changes would BEST resolve this issue?

A) Increase the Lambda function's timeout value and memory allocation only
B) Move the database connection creation outside the handler to the global scope, and place Amazon RDS Proxy in front of the database
C) Increase the RDS instance's max_connections parameter to a very high value
D) Add a dead-letter queue (DLQ) to the function to catch connection failures for later retry

---

**Q41.** A Lambda function is integrated with API Gateway using Lambda proxy integration. Clients intermittently receive HTTP 502 'Malformed Lambda proxy response' errors, although invoking the same function directly via the AWS CLI returns valid JSON. What is the MOST likely cause of this issue?

A) The API Gateway resource policy is denying the invoking IAM role
B) The API Gateway method's request validator is rejecting invalid input schemas
C) The Lambda function's reserved concurrency limit has been reached, causing throttling
D) The Lambda function occasionally returns a response object missing the required statusCode field, or returns the body as a non-string type

---

**Q42.** An application performs point lookups and query operations directly against a DynamoDB table using the DynamoDB API, and it needs microsecond-level read latency for frequently accessed items with minimal changes to existing application code. Which caching solution is BEST suited for this requirement?

A) Amazon ElastiCache for Redis with manual cache-aside logic added to the application
B) Amazon DynamoDB Accelerator (DAX) as a caching layer in front of the table
C) Amazon CloudFront caching in front of the API layer
D) Amazon ElastiCache for Memcached with TTL-based cache invalidation logic

---

**Q43.** A developer instruments a Lambda function with AWS X-Ray. They want to filter traces in the X-Ray console using a custom field called customerTier (values like 'gold' or 'silver'), and they also want to attach a large debug object containing the full request payload for offline analysis, without needing it to be searchable. How should the developer add these two pieces of data to the trace segment?

A) Add both customerTier and the debug payload as annotations
B) Add customerTier as an annotation and the debug payload as metadata
C) Add customerTier as metadata and the debug payload as an annotation
D) Add both customerTier and the debug payload as metadata

---

**Q44.** A flash-sale order-processing service stores orders in a DynamoDB table using on-demand capacity mode, partitioned by orderDate (format YYYY-MM-DD). During a flash sale, CloudWatch shows a spike in WriteThrottleEvents even though the table's total ConsumedWriteCapacityUnits is far below the account service quota and on-demand mode is enabled. What is the BEST long-term fix?

A) Redesign the partition key to include a higher-cardinality attribute (e.g., orderId or a random suffix) so writes are spread across more partitions
B) Switch the table to provisioned capacity mode with auto scaling enabled
C) Enable DynamoDB Accelerator (DAX) in front of the table to absorb the extra write traffic
D) Open an AWS Support case to manually raise the table's on-demand throughput ceiling before the sale

---

**Q45.** A team instruments a Lambda-based checkout function with the AWS X-Ray SDK. Support engineers need to filter traces in the X-Ray console to find every trace for a specific customerId, and they also want to attach the full request/response JSON payload (several KB) to each trace for offline debugging, without breaking trace filtering. Which approach correctly separates this data according to X-Ray's design?

A) Record the payload as an annotation and customerId as metadata
B) Record both customerId and the payload as annotations
C) Record customerId as an annotation and the payload as metadata
D) Record both customerId and the payload as metadata, since metadata supports console filter expressions

---

